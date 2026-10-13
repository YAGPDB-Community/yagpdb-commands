{{$config := sdict
    "profilePrefix" "rpg_profile_"

    "successColor" 3066993
    "errorColor" 15158332

    "footer" "World Boss System • Item Manager v1.0.0"
}}

{{/* =======================================================
   Admin Check
======================================================= */}}

{{if not (hasPermissions 8)}}

{{sendMessage nil (cembed
    "title" "❌ Access Denied"
    "description" "Only Administrators can use this command."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{/* =======================================================
   Usage
======================================================= */}}

{{if lt (len .CmdArgs) 4}}

{{sendMessage nil (cembed
    "title" "❌ Invalid Usage"
    "description" (print
"`!item give @user hp 10`\n"
"`!item remove @user hp 5`\n"
"`!item remove @user hp all`"
)
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Target User
======================================================= */}}

{{if lt (len .Message.Mentions) 1}}

{{sendMessage nil (cembed
    "title" "❌ Missing User"
    "description" "Mention a player."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{$target := index .Message.Mentions 0}}

{{/* =======================================================
   Parse Arguments
======================================================= */}}

{{$action := lower (index .CmdArgs 0)}}

{{$itemAlias := lower (index .CmdArgs 2)}}

{{$quantityArg := lower (index .CmdArgs 3)}}

{{$removeAll := false}}

{{$quantity := 1}}

{{if eq $quantityArg "all"}}

    {{$removeAll = true}}

{{else}}

    {{$quantity = toInt $quantityArg}}

{{end}}

{{/* =======================================================
   Load Profile
======================================================= */}}

{{$profileKey := print $config.profilePrefix $target.ID}}

{{$profileDB := dbGet $target.ID $profileKey}}

{{if not $profileDB}}

{{sendMessage nil (cembed
    "title" "❌ Profile Not Found"
    "description" "That player doesn't have an RPG profile."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{$profile := $profileDB.Value}}

{{if not $profile.inventory}}

    {{$profile.Set "inventory" (sdict)}}

{{end}}

{{$inventory := $profile.inventory}}

{{/* =======================================================
   Item Database
======================================================= */}}

{{$items := sdict}}

{{/* Consumables */}}

{{$items.Set "hp" (sdict
    "name" "HP Potion"
    "emoji" "❤️"
)}}

{{$items.Set "ghp" (sdict
    "name" "Greater HP Potion"
    "emoji" "💚"
)}}

{{$items.Set "maxhp" (sdict
    "name" "Max HP Potion"
    "emoji" "💖"
)}}

{{$items.Set "revive" (sdict
    "name" "Revive Scroll"
    "emoji" "💫"
)}}
{{/* Boosts */}}

{{$items.Set "atk" (sdict
    "name" "Attack Boost"
    "emoji" "⚔️"
)}}

{{$items.Set "def" (sdict
    "name" "Defense Boost"
    "emoji" "🛡️"
)}}

{{$items.Set "xp" (sdict
    "name" "XP Boost"
    "emoji" "⭐"
)}}

{{$items.Set "token" (sdict
    "name" "Token Boost"
    "emoji" "🪙"
)}}

{{/* Inventory */}}

{{$items.Set "bag5" (sdict
    "name" "Inventory Upgrade +5"
    "emoji" "🎒"
)}}

{{$items.Set "bag10" (sdict
    "name" "Inventory Upgrade +10"
    "emoji" "🎒"
)}}

{{$items.Set "bag20" (sdict
    "name" "Inventory Upgrade +20"
    "emoji" "🎒"
)}}


{{/* =======================================================
   Action Validation
======================================================= */}}

{{if and (ne $action "give") (ne $action "remove")}}

{{sendMessage nil (cembed
    "title" "❌ Invalid Action"
    "description" "Action must be **give** or **remove**."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{/* =======================================================
   Item Validation
======================================================= */}}

{{$item := index $items $itemAlias}}

{{if not $item}}

{{sendMessage nil (cembed
    "title" "❌ Unknown Item"
    "description" "This item does not exist."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{/* =======================================================
   Quantity Validation
======================================================= */}}

{{if and (not $removeAll) (lt $quantity 1)}}

{{sendMessage nil (cembed
    "title" "❌ Invalid Quantity"
    "description" "Quantity must be greater than **0**."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{if and (eq $action "give") $removeAll}}

{{sendMessage nil (cembed
    "title" "❌ Invalid Usage"
    "description" "`all` can only be used with **remove**."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{/* =======================================================
   Give / Remove Item
======================================================= */}}

{{$current := 0}}

{{with (index $inventory $itemAlias)}}
    {{$current = toInt .}}
{{end}}

{{if eq $action "give"}}

    {{$inventory.Set $itemAlias (add $current $quantity)}}

{{else}}

    {{if eq $current 0}}

        {{sendMessage nil (cembed
            "title" "❌ Item Not Found"
            "description" (print "<@" $target.ID "> doesn't own any **" $item.name "**.")
            "color" $config.errorColor
        )}}

        {{return}}

    {{end}}

    {{if $removeAll}}

        {{$quantity = $current}}

    {{end}}

    {{if gt $quantity $current}}

        {{sendMessage nil (cembed
            "title" "❌ Not Enough Items"
            "description" (printf
                "<@%d> only owns **%d × %s**."
                $target.ID
                $current
                $item.name
            )
            "color" $config.errorColor
        )}}

        {{return}}

    {{end}}

    {{if eq $quantity $current}}

        {{$inventory.Del $itemAlias}}

    {{else}}

        {{$inventory.Set $itemAlias (sub $current $quantity)}}

    {{end}}

{{end}}

{{/* =======================================================
   Save
======================================================= */}}

{{dbSet $target.ID $profileKey $profile}}

{{/* =======================================================
   Success Embed
======================================================= */}}

{{$title := "🎁 Item Given"}}

{{if eq $action "remove"}}
    {{$title = "🗑️ Item Removed"}}
{{end}}

{{$verb := "given to"}}

{{if eq $action "remove"}}
    {{$verb = "removed from"}}
{{end}}

{{sendMessage nil (cembed
    "title" $title

    "description" (print
        $item.emoji " **" $item.name "**\n\n"
        "📦 Quantity: **x" $quantity "**\n\n"
        "👤 Successfully " $verb " <@" $target.ID ">."
    )

    "color" $config.successColor

    "footer" (sdict
        "text" $config.footer
    )
)}}