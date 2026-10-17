
{{$config := sdict
    "leaderKey" "wb_leaderboard"
    "color" 16766720
    "footer" "World Boss System v2.0.0"
}}

{{$leaderDB := dbGet 0 $config.leaderKey}}

{{if not $leaderDB}}
    {{$embed := cembed
        "title" "🏆 World Boss Leaderboard"
        "description" "No players have attacked a World Boss yet!"
        "color" 15158332
        "footer" (sdict "text" $config.footer)
    }}
    {{sendMessage nil $embed}}
    {{return}}
{{end}}

{{$leader := $leaderDB.Value}}
{{$entries := cslice}}

{{range $id, $damage := $leader}}
    {{$entries = $entries.Append (sdict
        "id" $id
        "damage" (toInt $damage)
    )}}
{{end}}

{{/* Sort by damage, highest first */}}
{{$count := len $entries}}

{{range $i := seq 0 $count}}
    {{range $j := seq (add $i 1) $count}}
        {{$a := index $entries $i}}
        {{$b := index $entries $j}}

        {{if gt $b.damage $a.damage}}
            {{$entries.Set $i $b}}
            {{$entries.Set $j $a}}
        {{end}}
    {{end}}
{{end}}

{{$description := ""}}
{{$rank := 1}}

{{range $entries}}
    {{if le $rank 10}}
        {{$emoji := "🏅"}}

        {{if eq $rank 1}}
            {{$emoji = "🥇"}}
        {{else if eq $rank 2}}
            {{$emoji = "🥈"}}
        {{else if eq $rank 3}}
            {{$emoji = "🥉"}}
        {{end}}

        {{$description = print
            $description
            $emoji
            " **#"
            $rank
            "** <@"
            .id
            ">\n💥 Damage: **"
            .damage
            "**\n\n"
        }}

        {{$rank = add $rank 1}}
    {{end}}
{{end}}

{{$embed := cembed
    "title" "🏆 WORLD BOSS LEADERBOARD"
    "description" $description
    "color" $config.color
    "footer" (sdict "text" $config.footer)
}}

{{sendMessage nil $embed}}
