{{$config := sdict
    "profilePrefix" "rpg_profile_"

    "successColor" 3066993
    "errorColor" 15158332
    "embedColor" 3447003

    "footer" "World Boss System • Use v1.0.0"
}}

{{/* =======================================================
   Item Database
======================================================= */}}

{{$items := sdict}}

{{$items.Set "bag5" (sdict
    "name" "Inventory +5"
    "type" "inventory"
    "slots" 5
)}}

{{$items.Set "bag10" (sdict
    "name" "Inventory +10"
    "type" "inventory"
    "slots" 10
)}}

{{$items.Set "bag20" (sdict
    "name" "Inventory +20"
    "type" "inventory"
    "slots" 20
)}}

{{$items.Set "atk" (sdict
    "name" "Attack Boost"
    "emoji" "⚔️"
    "type" "boost"
)}}

{{$items.Set "def" (sdict
    "name" "Defense Boost"
    "emoji" "🛡️"
    "type" "boost"
)}}

{{$items.Set "xp" (sdict
    "name" "XP Boost"
    "emoji" "⭐"
    "type" "boost"
)}}

{{$items.Set "token" (sdict
    "name" "Boss Token Boost"
    "emoji" "🪙"
    "type" "boost"
)}}

{{/* =======================================================
   Usage
======================================================= */}}

{{if lt (len .CmdArgs) 1}}

{{sendMessage nil (cembed
    "title" "❌ Missing Item"
    "description" "Usage:\n`!use <item>`\n\nExample:\n`!use bag10`"
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{$itemAlias := lower (index .CmdArgs 0)}}

{{$item := index $items $itemAlias}}

{{if not $item}}

{{sendMessage nil (cembed
    "title" "❌ Unknown Item"
    "description" "This item cannot be used."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

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

{{$owned := 0}}

{{with (index $inventory $itemAlias)}}
    {{$owned = toInt .}}
{{end}}

{{if le $owned 0}}

{{sendMessage nil (cembed
    "title" "❌ Item Not Found"
    "description" (print "You don't own any **" $item.name "**.")
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{/* =======================================================
   Use Inventory Upgrade
======================================================= */}}

{{if eq $item.type "inventory"}}

    {{$currentSlots := or $profile.maxInventorySlots 20}}

    {{$newSlots := add $currentSlots $item.slots}}

    {{$profile.Set "maxInventorySlots" $newSlots}}

    {{if eq $owned 1}}

        {{$inventory.Del $itemAlias}}

    {{else}}

        {{$inventory.Set $itemAlias (sub $owned 1)}}

    {{end}}

    {{dbSet .User.ID $profileKey $profile}}

    {{sendMessage nil (cembed
        "title" "🎒 Inventory Expanded!"

        "description" (print
            "You used **" $item.name "**.\n\n"
            "📦 Inventory Slots: **"
            $currentSlots
            " → "
            $newSlots
            "**"
        )

        "color" $config.successColor

        "footer" (sdict
            "text" $config.footer
        )
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   Use Boosts
======================================================= */}}

{{if or
    (eq $itemAlias "atk")
    (eq $itemAlias "def")
    (eq $itemAlias "xp")
    (eq $itemAlias "token")
}}

    {{if not $profile.activeBoosts}}
        {{$profile.Set "activeBoosts" (sdict)}}
    {{end}}

    {{$boosts := $profile.activeBoosts}}

    {{$duration := 3600}}

    {{if or (eq $itemAlias "xp") (eq $itemAlias "token")}}
        {{$duration = 7200}}
    {{end}}

    {{$boosts.Set $itemAlias (currentTime.Add (toDuration (mult $duration 1000000000)))}}

    {{$current := toInt (index $inventory $itemAlias)}}

    {{if le $current 1}}
        {{$inventory.Del $itemAlias}}
    {{else}}
        {{$inventory.Set $itemAlias (sub $current 1)}}
    {{end}}

    {{dbSet .User.ID $profileKey $profile}}

    {{sendMessage nil (cembed
        "title" "⚡ Boost Activated"
        "description" (print
            "You activated **" $item.name "**!\n\n"
            "⏳ Duration: **"
            (div $duration 3600)
            " hour(s)**"
        )
        "color" $config.successColor
        "footer" (sdict
            "text" $config.footer
        )
    )}}

    {{return}}

{{end}}
{{/* =======================================================
   Fallback
======================================================= */}}

{{sendMessage nil (cembed
    "title" "❌ Cannot Use Item"
    "description" "This item cannot be used yet."
    "color" $config.errorColor
    "footer" (sdict
        "text" $config.footer
    )
)}}