{{$config := sdict

    "profilePrefix" "rpg_profile_"
    "statsPrefix" "wb_stats_"
    "cooldownPrefix" "wb_attack_cd_"

    "embedColor" 3447003
    "errorColor" 15158332

    "footer" "World Boss System v2.0.0"
}}


{{/* Admin Check */}}

{{if not (hasPermissions 8)}}

    {{sendMessage nil (cembed
        "title" "❌ Permission Denied"
        "description" "You need Administrator permission to use this command."
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}


{{/* Target Selection */}}

{{$targetID := .User.ID}}

{{if gt (len .Message.Mentions) 0}}

    {{$targetUser := index .Message.Mentions 0}}

    {{$targetID = $targetUser.ID}}

{{end}}


{{/* Profile Key */}}

{{$profileKey := print $config.profilePrefix $targetID}}

{{$profileDB := dbGet (toInt $targetID) $profileKey}}


{{if not $profileDB}}

    {{sendMessage nil (cembed
        "title" "❌ Profile Not Found"
        "description" (printf
            "<@%d> does not have an RPG profile."
            $targetID
        )
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}


{{/* Delete RPG Profile */}}

{{dbDel (toInt $targetID) $profileKey}}


{{/* Delete World Boss Stats */}}

{{$statsKey := print $config.statsPrefix $targetID}}

{{dbDel 0 $statsKey}}


{{/* Delete Attack Cooldown */}}

{{$cooldownKey := print $config.cooldownPrefix $targetID}}

{{dbDel 0 $cooldownKey}}


{{/* Confirmation */}}

{{$embed := cembed

    "title" "🗑️ RPG PROFILE RESET"

    "description" (printf
        "👤 Player: <@%d>\n\n✅ RPG Profile deleted\n✅ World Boss statistics deleted\n✅ Attack cooldown removed\n\nThe player can create a new profile using the RPG start command."
        $targetID
    )

    "color" $config.embedColor

    "footer" (sdict
        "text" $config.footer
    )
}}


{{sendMessage nil $embed}}