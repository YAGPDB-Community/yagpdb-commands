{{$config := sdict
    "bossKey" "wb_boss"
    "successColor" 3066993
    "errorColor" 15158332
    "footer" "World Boss System v2.0.0"
}}


{{/* Admin Check */}}

{{if not (hasPermissions 8)}}

    {{$embed := cembed
        "title" "❌ Permission Denied"
        "description" "You do not have permission to end a World Boss battle."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    }}

    {{sendMessage nil $embed}}
    {{return}}

{{end}}


{{/* Get Boss */}}

{{$bossDB := dbGet 0 $config.bossKey}}


{{if not $bossDB}}

    {{$embed := cembed
        "title" "⚠️ No Active Boss"
        "description" "There is no active World Boss battle to end."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    }}

    {{sendMessage nil $embed}}
    {{return}}

{{end}}


{{$boss := $bossDB.Value}}


{{/* Remove Active Boss */}}

{{dbDel 0 $config.bossKey}}


{{$embed := cembed
    "title" "🛑 WORLD BOSS ENDED"
    "description" (printf
        "The active battle has been closed.\n\n%s **%s**\n\n🆔 Battle ID: **#%d**\n\n👤 Ended by: <@%d>"
        $boss.emoji
        $boss.name
        $boss.battle_id
        .User.ID
    )
    "color" $config.successColor
    "footer" (sdict "text" $config.footer)
}}


{{sendMessage nil $embed}}