{{$config := sdict
    "profilePrefix" "rpg_profile_"
    "embedColor" 3447003
    "errorColor" 15158332
    "footer" "World Boss System • v2.0.0"
  
  "profileVersion" "2.1.0"

"defaultLevel" 1
"defaultXP" 0

"defaultHP" 1000
"defaultMaxHP" 1000

"defaultAttack" 10
"defaultDefense" 5

"defaultTokens" 0

"defaultInventorySlots" 20
}}

{{/* Target User */}}

{{$targetID := .User.ID}}

{{if gt (len .Message.Mentions) 0}}
    {{$targetID = (index .Message.Mentions 0).ID}}
{{end}}

{{/* Load Profile */}}

{{$profileKey := print $config.profilePrefix $targetID}}

{{$now := currentTime.Unix}}

{{$profileDB := dbGet (toInt $targetID) $profileKey}}

{{if not $profileDB}}

{{$profile := sdict}}

{{$profile.Set "profileVersion" $config.profileVersion}}

{{$profile.Set "level" $config.defaultLevel}}
{{$profile.Set "xp" $config.defaultXP}}

{{$profile.Set "hp" $config.defaultHP}}
{{$profile.Set "maxHP" $config.defaultMaxHP}}

{{$profile.Set "attack" $config.defaultAttack}}
{{$profile.Set "defense" $config.defaultDefense}}

{{$profile.Set "tokens" $config.defaultTokens}}

{{$profile.Set "alive" true}}

{{$profile.Set "inventory" (sdict
    "hp" 3
)}}

{{$profile.Set "maxInventorySlots" $config.defaultInventorySlots}}

{{$profile.Set "equipment" (sdict
    "weapon" ""
    "armor" ""
    "shield" ""
)}}

{{$profile.Set "activeBoosts" (sdict)}}

{{$profile.Set "statistics" (sdict
    "totalBossDamage" 0
    "bossesKilled" 0
    "attacks" 0
    "deaths" 0
    "potionsUsed" 0
)}}

{{$profile.Set "createdAt" $now}}
{{$profile.Set "lastSeen" $now}}

{{dbSet (toInt $targetID) $profileKey $profile}}

{{sendMessage nil (cembed
    "title" "⚔️ Welcome to World Boss System!"
    "description" (print
"Your profile has been created successfully!\n\n"

"🎁 **Starter Pack Received**\n"
"❤️ HP Potion ×3\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"📖 Use **!boss** to view the current World Boss.\n"
"⚔️ Use **!attack** to join the battle.\n\n"

"Good luck, Hero!"
    )
    "color" $config.successColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{$profile := $profileDB.Value}}

{{/* XP Progress */}}

{{$level := $profile.level}}
{{$currentXP := $profile.xp}}

{{/* Change this if your level system uses another formula */}}
{{$requiredXP := mult $level 100}}

{{if lt $requiredXP 1}}
    {{$requiredXP = 1}}
{{end}}

{{$xpPercent := div (mult $currentXP 100) $requiredXP}}

{{if gt $xpPercent 100}}
    {{$xpPercent = 100}}
{{end}}

{{$filled := div $xpPercent 10}}

{{$xpBar := ""}}

{{range seq 1 10}}

    {{if le . $filled}}
        {{$xpBar = print $xpBar "█"}}
    {{else}}
        {{$xpBar = print $xpBar "□"}}
    {{end}}

{{end}}
{{/* Profile Embed */}}

{{$embed := cembed
    "title" "⚔️ RPG PROFILE"

    "fields" (cslice

    (sdict
        "name" "👤 Player"
        "value" (print "<@" $targetID ">")
        "inline" false
    )

    (sdict
        "name" "⭐ Level"
        "value" (print $profile.level)
        "inline" true
    )

    (sdict
        "name" "❤️ HP"
        "value" (printf "%d / %d" $profile.hp $profile.maxHP)
        "inline" true
    )

    (sdict
        "name" "🪙 Boss Tokens"
        "value" (print $profile.tokens)
        "inline" true
    )

    (sdict
        "name" "⚔️ Attack"
        "value" (print $profile.attack)
        "inline" true
    )

    (sdict
        "name" "🛡️ Defense"
        "value" (print $profile.defense)
        "inline" true
    )

    (sdict
        "name" "‎"
        "value" "‎"
        "inline" true
    )

    (sdict
        "name" "📈 Experience"
        "value" (printf "%s **%d%%**\n\n**%d / %d XP**"
            $xpBar
            $xpPercent
            $currentXP
            $requiredXP
        )
        "inline" false
    )

)

    "color" $config.embedColor

    "footer" (sdict
        "text" $config.footer
    )
}}

{{sendMessage nil $embed}}