{{$config := sdict
    "bossKey" "wb_boss"
    "battleKey" "wb_battle_id"
    "embedColor" 16711680
    "footer" "World Boss System v2.0.0"
}}

{{/* Admin Check */}}
{{if not (hasPermissions 8)}}
    {{sendMessage nil "❌ You do not have permission to spawn a World Boss."}}
    {{return}}
{{end}}


{{/* Check Existing Boss */}}
{{$existing := dbGet 0 $config.bossKey}}

{{if $existing}}
    {{$boss := $existing.Value}}

    {{if $boss.alive}}
        {{sendMessage nil "⚠️ There is already an active World Boss!"}}
        {{return}}
    {{end}}
{{end}}


{{/* Boss Pool */}}

{{$bosses := cslice
    (sdict
        "id" "dragon_001"
        "name" "Ancient Dragon"
        "emoji" "🐉"
        "hp" 5000
        "max_hp" 5000
        "alive" true
        "min_attack" 30
        "max_attack" 60
        "critChance" 5
        "critMultiplier" 2
    )
    (sdict
        "id" "demon_001"
        "name" "Demon King"
        "emoji" "👹"
        "hp" 7500
        "max_hp" 7500
        "min_attack" 50
        "max_attack" 90
        "critChance" 8
        "critMultiplier" 2
        "alive" true
    )
    (sdict
        "id" "kraken_001"
        "name" "Kraken"
        "emoji" "🌊"
        "hp" 8000
        "max_hp" 8000
        "alive" true
        "min_attack" 80
        "max_attack" 130
        "critChance" 10
        "critMultiplier" 2
    )
    (sdict
        "id" "titan_001"
        "name" "Mecha Titan"
        "emoji" "🤖"
        "hp" 10000
        "max_hp" 10000
         "min_attack" 120
         "max_attack" 180
         "critChance" 15
         "critMultiplier" 2
         "alive" true
    )
}}


{{/* Random Boss */}}

{{$random := randInt 0 (len $bosses)}}
{{$selected := index $bosses $random}}


{{/* Battle ID */}}

{{$oldID := dbGet 0 $config.battleKey}}

{{$battleID := 1}}

{{if $oldID}}
    {{$battleID = add (toInt $oldID.Value) 1}}
{{end}}

{{dbSet 0 $config.battleKey $battleID}}


{{/* Create Boss Data */}}

{{$bossData := sdict
    "id" $selected.id
    "battle_id" $battleID
    "name" $selected.name
    "emoji" $selected.emoji
    "hp" $selected.hp
    "max_hp" $selected.hp
    "min_attack" $selected.min_attack
    "max_attack" $selected.max_attack
    "critChance" $selected.critChance
    "critMultiplier" $selected.critMultiplier
    "alive" true
    "spawned_by" .User.ID
    "spawned_at" currentTime.Unix
}}


{{dbSet 0 $config.bossKey $bossData}}


{{/* Announcement */}}

{{$embed := cembed
    "title" "🌍 WORLD BOSS HAS APPEARED!"
    "description" (printf
        "%s **%s**\n\n❤️ HP: **%d / %d**\n⚔️ Damage: **%d - %d**\n\n⚔️ Use `!attack` to join the battle!\n\n🆔 Battle ID: **#%d**"
        $selected.emoji
        $selected.name
        $selected.hp
        $selected.hp
        $selected.min_attack
        $selected.max_attack
        $battleID
    )
    "color" $config.embedColor
    "footer" (sdict "text" $config.footer)
}}

{{sendMessage nil $embed}}