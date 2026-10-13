{{$config := sdict

    "version" 1

    "profilePrefix" "rpg_profile_"

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


{{/* Arguments Check */}}

{{if lt (len .Args) 3}}

    {{sendMessage nil (cembed
        "title" "❌ Incorrect Usage"
        "description" "Usage:\n`!setstat @user <stat> <value>`"
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}



{{/* Target User */}}

{{if lt (len .Message.Mentions) 1}}

    {{sendMessage nil (cembed
        "title" "❌ Invalid User"
        "description" "Please mention a valid user."
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}

{{$targetUser := index .Message.Mentions 0}}

{{$targetID := $targetUser.ID}}



{{/* Stat & Value */}}

{{$stat := ""}}

{{range .Args}}

    {{if in (cslice "level" "xp" "hp" "maxhp" "attack" "defense" "tokens") (lower .)}}

        {{$stat = lower .}}

    {{end}}

{{end}}

{{$value := ""}}

{{range .Args}}

    {{if and (ne (lower .) $stat) (reFind `^[0-9]+$` .)}}

        {{$value = .}}

    {{end}}

{{end}}



{{/* Load Profile */}}

{{$profileKey := print $config.profilePrefix $targetID}}

{{$profileDB := dbGet (toInt $targetID) $profileKey}}



{{if not $profileDB}}

    {{sendMessage nil (cembed
        "title" "❌ Profile Not Found"
        "description" "This user does not have an RPG profile yet."
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}



{{$profile := $profileDB.Value}}



{{/* Allowed Stats */}}

{{$validStats := cslice
    "level"
    "xp"
    "hp"
    "maxhp"
    "attack"
    "defense"
    "tokens"
}}



{{if not (in $validStats $stat)}}

    {{sendMessage nil (cembed
        "title" "❌ Invalid Stat"
        "description" (print
            "Available stats:\n"
            "`level`\n"
            "`xp`\n"
            "`hp`\n"
            "`maxhp`\n"
            "`attack`\n"
            "`defense`\n"
            "`tokens`"
        )
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}

{{/* Value Conversion */}}

{{$oldValue := ""}}
{{$newValue := 0}}


{{/* Special HP max */}}

{{if and (eq $stat "hp") (eq (lower (str $value)) "max")}}

    {{$oldValue = print $profile.hp}}

    {{$profile.Set "hp" $profile.maxHP}}
    {{$profile.Set "alive" true}}

    {{$newValue = $profile.maxHP}}


{{else}}


    {{$number := toInt $value}}

    {{if lt $number 0}}

        {{sendMessage nil (cembed
            "title" "❌ Invalid Value"
            "description" "Values cannot be negative."
            "color" $config.errorColor
        )}}

        {{return}}

    {{end}}



    {{/* Get Old Value */}}

    {{if eq $stat "level"}}
        {{$oldValue = print $profile.level}}

    {{else if eq $stat "xp"}}
        {{$oldValue = print $profile.xp}}

    {{else if eq $stat "hp"}}
        {{$oldValue = print $profile.hp}}

    {{else if eq $stat "maxhp"}}
        {{$oldValue = print $profile.maxHP}}

    {{else if eq $stat "attack"}}
        {{$oldValue = print $profile.attack}}

    {{else if eq $stat "defense"}}
        {{$oldValue = print $profile.defense}}

    {{else if eq $stat "tokens"}}
        {{$oldValue = print $profile.tokens}}

    {{end}}



    {{/* Set New Value */}}

    {{if eq $stat "level"}}

        {{$profile.Set "level" $number}}
        {{$newValue = $number}}


    {{else if eq $stat "xp"}}

        {{$profile.Set "xp" $number}}
        {{$newValue = $number}}


    {{else if eq $stat "hp"}}

        {{$profile.Set "hp" $number}}

        {{if eq $number 0}}
            {{$profile.Set "alive" false}}
        {{else}}
            {{$profile.Set "alive" true}}
        {{end}}

        {{$newValue = $number}}


    {{else if eq $stat "maxhp"}}

        {{$profile.Set "maxHP" $number}}

        {{if gt $profile.hp $number}}
            {{$profile.Set "hp" $number}}
        {{end}}

        {{$newValue = $number}}


    {{else if eq $stat "attack"}}

        {{$profile.Set "attack" $number}}
        {{$newValue = $number}}


    {{else if eq $stat "defense"}}

        {{$profile.Set "defense" $number}}
        {{$newValue = $number}}


    {{else if eq $stat "tokens"}}

        {{$profile.Set "tokens" $number}}
        {{$newValue = $number}}

    {{end}}


{{end}}



{{/* Save Profile */}}

{{dbSet (toInt $targetID) $profileKey $profile}}



{{/* Confirmation Embed */}}

{{$embed := cembed

    "title" "🛠️ RPG STAT UPDATED"

    "description" (printf
        "👤 Player: <@%d>\n\n📊 Stat: **%s**\n\n📉 Previous: **%s**\n📈 New: **%d**"
        (toInt $targetID)
        $stat
        $oldValue
        $newValue
    )

    "color" $config.embedColor

    "footer" (sdict
        "text" $config.footer
    )
}}


{{sendMessage nil $embed}}