{{$config := sdict
    "bossKey" "wb_boss"
    "embedColor" 15105570
    "errorColor" 15158332
    "footer" "World Boss System v2.0.0"
}}


{{/* Get Active Boss */}}

{{$bossData := dbGet 0 $config.bossKey}}


{{if not $bossData}}

    {{$embed := cembed
        "title" "🌍 No Active World Boss"
        "description" "There is currently no World Boss battle active."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    }}

    {{sendMessage nil $embed}}
    {{return}}

{{end}}


{{$boss := $bossData.Value}}


{{/* Check Boss Status */}}

{{if not $boss.alive}}

    {{$embed := cembed
        "title" "🏁 Boss Battle Finished"
        "description" "The previous World Boss has already been defeated."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    }}

    {{sendMessage nil $embed}}
    {{return}}

{{end}}


{{/* Calculate HP Percentage */}}

{{$percentage := mult (fdiv $boss.hp $boss.max_hp) 100}}


{{$embed := cembed
    "title" "🌍 ACTIVE WORLD BOSS"
    "description" (printf
        "%s **%s**\n\n❤️ HP: **%d / %d**\n📊 Health: **%.0f%%**\n\n🆔 Battle ID: **#%d**\n\n⚔️ Use `!attack` to fight!"
        $boss.emoji
        $boss.name
        $boss.hp
        $boss.max_hp
        $percentage
        $boss.battle_id
    )
    "color" $config.embedColor
    "footer" (sdict "text" $config.footer)
}}


{{sendMessage nil $embed}}