{{$config := sdict
    "profilePrefix" "rpg_profile_"

    "successColor" 3066993
    "errorColor" 15158332
    "embedColor" 3447003

    "footer" "World Boss System • Sell v1.0.0"
}}

{{/* =======================================================
   Load Profile
======================================================= */}}

{{$profileKey := print $config.profilePrefix .User.ID}}

{{$profileDB := dbGet .User.ID $profileKey}}

{{if not $profileDB}}

{{sendMessage nil (cembed
    "title" "❌ No RPG Profile"
    "description" "Create your profile first with **!profile**."
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

{{$items.Set "hp" (sdict
    "name" "HP Potion"
    "emoji" "❤️"
    "price" 25
    "sellPrice" 12
    "category" "consumable"
    "stackable" true
    "sellable" true
)}}

{{$items.Set "ghp" (sdict
    "name" "Greater HP Potion"
    "emoji" "💚"
    "price" 60
    "sellPrice" 30
    "category" "consumable"
    "stackable" true
    "sellable" true
)}}

{{$items.Set "maxhp" (sdict
    "name" "Max HP Potion"
    "emoji" "💖"
    "price" 120
    "sellPrice" 60
    "category" "consumable"
    "stackable" true
    "sellable" true
)}}

{{/* =======================================================
   Missing Arguments
======================================================= */}}

{{if lt (len .CmdArgs) 1}}

{{sendMessage nil (cembed
    "title" "❌ Missing Item"
    "description" "Usage:\n`!sell <item> [quantity]`\n\nExamples:\n`!sell hp`\n`!sell hp 5`\n`!sell hp all`\n`!sell all hp`"
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Parse Arguments
======================================================= */}}

{{$itemAlias := ""}}
{{$quantity := 1}}
{{$sellAll := false}}

{{if eq (lower (index .CmdArgs 0)) "all"}}

    {{$sellAll = true}}
    {{$itemAlias = lower (index .CmdArgs 1)}}

{{else}}

    {{$itemAlias = lower (index .CmdArgs 0)}}

    {{if ge (len .CmdArgs) 2}}

        {{if eq (lower (index .CmdArgs 1)) "all"}}

            {{$sellAll = true}}

        {{else}}

            {{$quantity = toInt (index .CmdArgs 1)}}

        {{end}}

    {{end}}

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
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Sellable Check
======================================================= */}}

{{if not $item.sellable}}

{{sendMessage nil (cembed
    "title" "❌ Item Cannot Be Sold"
    "description" (print "**" $item.name "** cannot be sold.")
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Inventory Check
======================================================= */}}

{{$owned := 0}}

{{with (index $inventory $itemAlias)}}
    {{$owned = toInt .}}
{{end}}

{{if le $owned 0}}

{{sendMessage nil (cembed
    "title" "❌ Item Not Found"
    "description" (print "You don't have any **" $item.name "**.")
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Quantity
======================================================= */}}

{{if $sellAll}}
    {{$quantity = $owned}}
{{end}}

{{if lt $quantity 1}}

{{sendMessage nil (cembed
    "title" "❌ Invalid Quantity"
    "description" "Quantity must be greater than **0**."
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{if gt $quantity $owned}}

{{sendMessage nil (cembed
    "title" "❌ Not Enough Items"
    "description" (printf
"You only own **%d × %s**."
$owned
$item.name
)
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   Calculate Reward
======================================================= */}}

{{$reward := mult $item.sellPrice $quantity}}

{{/* =======================================================
   Remove Items
======================================================= */}}

{{if le $quantity $owned}}

    {{if eq $quantity $owned}}

        {{$inventory.Del $itemAlias}}

    {{else}}

        {{$inventory.Set $itemAlias (sub $owned $quantity)}}

    {{end}}

{{end}}

{{/* =======================================================
   Give Boss Tokens
======================================================= */}}

{{$profile.Set "tokens" (add $profile.tokens $reward)}}

{{/* =======================================================
   Save Profile
======================================================= */}}

{{dbSet .User.ID $profileKey $profile}}

{{/* =======================================================
   Success Embed
======================================================= */}}

{{sendMessage nil (cembed
    "title" "💰 Item Sold"

    "description" (print
        $item.emoji " **" $item.name "**\n\n"
        "📦 Sold: **x" $quantity "**\n\n"
        "🪙 Earned: **+" $reward " Boss Tokens**\n\n"
        "💰 Current Balance: **" $profile.tokens " Boss Tokens**"
    )

    "color" $config.successColor

    "footer" (sdict
        "text" $config.footer
    )
)}}