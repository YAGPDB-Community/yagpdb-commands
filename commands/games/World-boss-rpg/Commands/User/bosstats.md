{{$config := sdict
    "statsPrefix" "wb_stats_"
    "successColor" 3066993
    "errorColor" 15158332
    "footer" "World Boss System v2.0.0"
}}


{{$target := .User}}

{{if .Message.Mentions}}
    {{$target = index .Message.Mentions 0}}
{{end}}

{{$statsKey := print $config.statsPrefix $target.ID}}

{{$statsDB := dbGet 0 $statsKey}}


{{if not $statsDB}}

    {{$embed := cembed
        "title" "📊 No Statistics Found"
        "description" "You have not attacked any World Boss yet!"
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    }}

    {{sendMessage nil $embed}}
    {{return}}

{{end}}


{{$stats := $statsDB.Value}}


{{$embed := cembed
    "title" "📊 World Boss Statistics"
    "description" (printf
        "👤 Player: <@%d>\n\n⚔️ Total Attacks: **%d**\n💥 Total Damage: **%d**\n🔥 Critical Hits: **%d**\n🏹 Best Hit: **%d**"
        $target.ID
        $stats.attacks
        $stats.damage
        $stats.crits
        $stats.best_hit
    )
    "color" $config.successColor
    "footer" (sdict "text" $config.footer)
}}


{{sendMessage nil $embed}}