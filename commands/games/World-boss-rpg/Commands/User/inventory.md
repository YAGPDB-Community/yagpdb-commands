{{$config := sdict

    "profilePrefix" "rpg_profile_"

    "defaultSlots" 20

    "embedColor" 3447003
    "errorColor" 15158332

    "footer" "World Boss System • Inventory v2.1.0"

}}

{{/* =======================================================
   Target User
======================================================= */}}

{{$targetID := .User.ID}}

{{if gt (len .Message.Mentions) 0}}
    {{$targetID = (index .Message.Mentions 0).ID}}
{{end}}

{{/* =======================================================
   Load Profile
======================================================= */}}

{{$profileKey := print $config.profilePrefix $targetID}}

{{$profileDB := dbGet (toInt $targetID) $profileKey}}

{{if not $profileDB}}

{{sendMessage nil (cembed
    "title" "❌ Profile Not Found"
    "description" "Create your profile first using **!profile**."
    "color" $config.errorColor
)}}

{{return}}

{{end}}

{{$profile := $profileDB.Value}}

{{if not $profile.inventory}}
    {{$profile.Set "inventory" (sdict)}}
{{end}}

{{$inventory := $profile.inventory}}

{{$maxSlots := or $profile.maxInventorySlots $config.defaultSlots}}

{{/* =======================================================
   Item Database
======================================================= */}}

{{$itemDB := sdict}}

{{/* =========================
   Consumables
========================= */}}

{{$itemDB.Set "hp" (sdict
    "name" "HP Potion"
    "emoji" "❤️"
    "category" "consumable"
)}}

{{$itemDB.Set "ghp" (sdict
    "name" "Greater HP Potion"
    "emoji" "💚"
    "category" "consumable"
)}}

{{$itemDB.Set "maxhp" (sdict
    "name" "Max HP Potion"
    "emoji" "💖"
    "category" "consumable"
)}}

{{$itemDB.Set "revive" (sdict
    "name" "Revive Scroll"
    "emoji" "💫"
    "category" "consumable"
)}}

{{/* =========================
   Boosts
========================= */}}

{{$itemDB.Set "atk" (sdict
    "name" "Attack Boost"
    "emoji" "⚔️"
    "category" "boost"
)}}

{{$itemDB.Set "def" (sdict
    "name" "Defense Boost"
    "emoji" "🛡️"
    "category" "boost"
)}}

{{$itemDB.Set "xp" (sdict
    "name" "XP Boost"
    "emoji" "⭐"
    "category" "boost"
)}}

{{$itemDB.Set "token" (sdict
    "name" "Boss Token Boost"
    "emoji" "🪙"
    "category" "boost"
)}}

{{/* =========================
   Inventory Upgrades
========================= */}}

{{$itemDB.Set "bag5" (sdict
    "name" "Inventory +5"
    "emoji" "🎒"
    "category" "inventory"
)}}

{{$itemDB.Set "bag10" (sdict
    "name" "Inventory +10"
    "emoji" "🎒"
    "category" "inventory"
)}}

{{$itemDB.Set "bag20" (sdict
    "name" "Inventory +20"
    "emoji" "🎒"
    "category" "inventory"
)}}

{{/* =======================================================
   Inventory Parsing
======================================================= */}}

{{$usedSlots := 0}}

{{$consumables := ""}}
{{$boosts := ""}}
{{$inventoryItems := ""}}

{{range $itemID, $amount := $inventory}}

    {{if gt (toInt $amount) 0}}

        {{$usedSlots = add $usedSlots 1}}

        {{$item := index $itemDB $itemID}}

        {{$emoji := "📦"}}
        {{$name := $itemID}}
        {{$category := ""}}

        {{if $item}}
            {{$emoji = $item.emoji}}
            {{$name = $item.name}}
            {{$category = $item.category}}
        {{end}}

        {{$line := print
            $emoji
            " "
            $name
            " ×"
            $amount
            "\n"
        }}

        {{if eq $category "consumable"}}

            {{$consumables = print $consumables $line}}

        {{else if eq $category "boost"}}

            {{$boosts = print $boosts $line}}

        {{else if eq $category "inventory"}}

            {{$inventoryItems = print $inventoryItems $line}}

        {{end}}

    {{end}}

{{end}}

{{if eq $consumables ""}}
    {{$consumables = "*Empty*"}}
{{end}}

{{if eq $boosts ""}}
    {{$boosts = "*Empty*"}}
{{end}}

{{if eq $inventoryItems ""}}
    {{$inventoryItems = "*Empty*"}}
{{end}}

{{$freeSlots := sub $maxSlots $usedSlots}}

{{$capacityText := print
"📦 Used: **" $usedSlots " / " $maxSlots "**\n"
"📭 Free: **" $freeSlots "**"
}}

{{/* =======================================================
   Inventory Embed
======================================================= */}}

{{$embed := cembed
    "title" "🎒 INVENTORY"

    "fields" (cslice

        (sdict
            "name" "👤 Player"
            "value" (print "<@" $targetID ">")
            "inline" false
        )

        (sdict
    "name" "❤️ Consumables"
    "value" $consumables
    "inline" false
)

(sdict
    "name" "⚡ Boosts"
    "value" $boosts
    "inline" false
)

(sdict
    "name" "🎒 Inventory Upgrades"
    "value" $inventoryItems
    "inline" false
)

              (sdict
    "name" "📊 Capacity"
    "value" $capacityText
    "inline" false
)

    )

    "color" $config.embedColor

    "footer" (sdict
        "text" $config.footer
    )
}}

{{sendMessage nil $embed}}