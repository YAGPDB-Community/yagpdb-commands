{{$config := sdict
"bossKey" "wb_boss"
"statsPrefix" "wb_stats_"
"minDamage" 20  
"maxDamage" 50  
"critChance" 10  
"critMultiplier" 2  
"missChance" 5  
"successColor" 3066993  
"errorColor" 15158332  
"footer" "World Boss System v2.0.0"
}}
{{$profileKey := print "rpg_profile_" .User.ID}}
{{$profileDB := dbGet .User.ID $profileKey}}
{{if not $profileDB}}
{{sendMessage nil (cembed
"title" "❌ No RPG Profile"
"description" "Create your profile first with !profile."
"color" $config.errorColor
)}}
{{return}}
{{end}}
{{$profile := $profileDB.Value}}
{{/* Active Attack Boost */}}
{{$atkBoost := false}}

{{if $profile.activeBoosts}}

{{$boosts := $profile.activeBoosts}}  

{{with index $boosts "atk"}}  

    {{if gt .Unix (currentTime.Unix)}}  

        {{$atkBoost = true}}  

    {{else}}  

        {{$boosts.Del "atk"}}  
        {{$profile.Set "activeBoosts" $boosts}}  

    {{end}}  

{{end}}

{{end}}

{{/* Active Defence boost*/}}

{{$defBoost := false}}

{{if $profile.activeBoosts}}

{{$boosts := $profile.activeBoosts}}  

{{with index $boosts "def"}}  

    {{if gt .Unix (currentTime.Unix)}}  

        {{$defBoost = true}}  

    {{else}}  

        {{$boosts.Del "def"}}  
        {{$profile.Set "activeBoosts" $boosts}}  

    {{end}}  

{{end}}

{{end}}

{{/* Active Token Boost */}}

{{$tokenBoost := false}}

{{if $profile.activeBoosts}}

{{$boosts := $profile.activeBoosts}}  

{{with index $boosts "token"}}  

    {{if gt .Unix (currentTime.Unix)}}  

        {{$tokenBoost = true}}  

    {{else}}  

        {{$boosts.Del "token"}}  
        {{$profile.Set "activeBoosts" $boosts}}  

    {{end}}  

{{end}}

{{end}}

{{/* Active XP Boost */}}

{{$xpBoost := false}}

{{if $profile.activeBoosts}}
{{$boosts := $profile.activeBoosts}}  
{{with index $boosts "xp"}}  
    {{if gt .Unix (currentTime.Unix)}}  
        {{$xpBoost = true}}  
    {{else}}  
        {{$boosts.Del "xp"}}  
        {{$profile.Set "activeBoosts" $boosts}}  
    {{end}}  
{{end}}
{{end}}
{{/* Attack Cooldown */}}
{{$cooldownKey := print "wb_attack_cd_" .User.ID}}
{{if not (hasPermissions 8)}}
{{$cd := dbGet 0 $cooldownKey}}
{{if $cd}}
{{$expires := $cd.ExpiresAt.Unix}}  
{{sendMessage nil (cembed  
    "title" "⏳ ATTACK COOLDOWN"  
    "description" (printf  
        "⚔️ You are recovering from your previous attack.\n\n🕒 You can attack again **<t:%d:R>**."  
        $expires  
    )  
    "color" $config.errorColor  
    "footer" (sdict  
        "text" $config.footer  
    )  
)}}  
{{return}}
{{end}}
{{dbSetExpire 0 $cooldownKey true 600}}
{{end}}
{{if not $profile.alive}}
{{sendMessage nil (cembed  
    "title" "💀 You are dead!"  
    "description" "Wait until someone revives you before attacking again."  
    "color" $config.errorColor  
)}}  
{{return}}
{{end}}
{{/* Get Active Boss */}}
{{$bossDB := dbGet 0 $config.bossKey}}
{{if not $bossDB}}
{{$embed := cembed  
    "title" "❌ No Active World Boss"  
    "description" "There is currently no boss to attack."  
    "color" $config.errorColor  
    "footer" (sdict "text" $config.footer)  
}}  
{{sendMessage nil $embed}}  
{{return}}
{{end}}
{{$boss := $bossDB.Value}}
{{if not $boss.alive}}
{{$embed := cembed  
    "title" "🏁 Battle Finished"  
    "description" "This World Boss has already been defeated."  
    "color" $config.errorColor  
    "footer" (sdict "text" $config.footer)  
}}  

{{sendMessage nil $embed}}  
{{return}}

{{end}}

{{/* Miss Check */}}

{{$missRoll := randInt 1 101}}

{{if le $missRoll $config.missChance}}

{{$embed := cembed  
    "title" "❌ Attack Missed!"  
    "description" (printf  
        "<@%d> attacked **%s %s** but missed!"  
        .User.ID  
        $boss.emoji  
        $boss.name  
    )  
    "color" $config.errorColor  
    "footer" (sdict "text" $config.footer)  
}}  

{{sendMessage nil $embed}}  
{{return}}

{{end}}

{{/* Damage Calculation */}}

{{$damage := add
$profile.attack
(randInt 0 21)
}}

{{if $atkBoost}}

{{$damage = div (mult $damage 125) 100}}

{{end}}

{{$critical := false}}

{{$critRoll := randInt 1 101}}

{{if le $critRoll $config.critChance}}

{{$damage = mult $damage $config.critMultiplier}}  
{{$critical = true}}

{{end}}

{{/* Update Boss HP */}}

{{$newHP := sub $boss.hp $damage}}

{{if lt $newHP 0}}
{{$newHP = 0}}
{{end}}

{{$boss.Set "hp" $newHP}}

{{if eq $newHP 0}}
{{$boss.Set "alive" false}}
{{end}}

{{dbSet 0 $config.bossKey $boss}}

{{$bossMessages := cslice
"attacks fiercely!"
"unleashes a devastating strike!"
"charges toward you!"
"launches a powerful attack!"
"uses a special ability!"
"strikes without mercy!"
}}

{{$bossAction := index $bossMessages (randInt 0 (len $bossMessages))}}

{{/* Boss Counter Attack */}}

{{$bossHit := randInt $boss.min_attack (add $boss.max_attack 1)}}

{{$bossCritical := false}}

{{if le (randInt 1 101) $boss.critChance}}
{{$bossHit = mult $bossHit $boss.critMultiplier}}
{{$bossCritical = true}}
{{end}}

{{$bossDamage := sub $bossHit $profile.defense}}

{{if lt $bossDamage 1}}
{{$bossDamage = 1}}
{{end}}
{{if $defBoost}}
{{$bossDamage = div (mult $bossDamage 75) 100}}
{{end}}

{{if lt $bossDamage 1}}
{{$bossDamage = 1}}
{{end}}

{{$newPlayerHP := sub $profile.hp $bossDamage}}

{{if lt $newPlayerHP 0}}
{{$newPlayerHP = 0}}
{{end}}

{{$profile.Set "hp" $newPlayerHP}}
{{$tokenReward := 1}}

{{if $critical}}
{{$tokenReward = 5}}
{{end}}
{{if $tokenBoost}}
    {{$tokenReward = mult $tokenReward 2}}
{{end}}

{{$profile.Set "tokens" (add $profile.tokens $tokenReward)}}

{{$xpReward := randInt 5 13}}

{{if $critical}}
{{$xpReward = add $xpReward 15}}
{{end}}
{{if $xpBoost}}
    {{$xpReward = add $xpReward 10}}
{{end}}
{{$newXP := add $profile.xp $xpReward}}
{{$profile.Set "xp" $newXP}}

{{$levelUp := false}}

{{$neededXP := mult $profile.level 100}}

{{if ge $newXP $neededXP}}

{{$profile.Set "xp" (sub $newXP $neededXP)}} 

{{$profile.Set "level" (add $profile.level 1)}}  

{{$profile.Set "maxHP" (add $profile.maxHP 50)}}  

{{$profile.Set "hp" $profile.maxHP}}  

{{$profile.Set "attack" (add $profile.attack 2)}}  

{{$profile.Set "defense" (add $profile.defense 1)}}  

{{$levelUp = true}}

{{end}}

{{/* Death Check */}}

{{$deathMessage := ""}}

{{if and (eq $newPlayerHP 0) (not $levelUp)}}

{{$profile.Set "alive" false}}  

{{$deathMessage = "\n\n☠️ **You have been defeated by the World Boss!**\nSomeone else has to use `!revive @user` to revive you."}}

{{end}}

{{dbSet .User.ID $profileKey $profile}}

{{/* Player Stats */}}

{{$statsKey := print $config.statsPrefix .User.ID}}

{{$statsDB := dbGet 0 $statsKey}}

{{if $statsDB}}

{{$stats := $statsDB.Value}}  

{{$stats.Set "damage" (add $stats.damage $damage)}}

{{$stats.Set "attacks" (add $stats.attacks 1)}}

{{if $critical}}  
    {{$stats.Set "crits" (add $stats.crits 1)}}  
{{end}}  

{{if gt $damage $stats.best_hit}}  
    {{$stats.Set "best_hit" $damage}}  
{{end}}  

{{dbSet 0 $statsKey $stats}}

{{else}}

{{$critCount := 0}}

{{if $critical}}
{{$critCount = 1}}
{{end}}

{{$stats := sdict
"damage" $damage
"attacks" 1
"crits" $critCount
"best_hit" $damage
}}

{{dbSet 0 $statsKey $stats}}

{{end}}

{{$leaderKey := "wb_leaderboard"}}

{{$leaderDB := dbGet 0 $leaderKey}}

{{if $leaderDB}}
{{$leader := $leaderDB.Value}}

{{$current := toInt (index $leader (str .User.ID))}}  

{{$leader.Set (str .User.ID) (add $current $damage)}}  

{{dbSet 0 $leaderKey $leader}}

{{else}}

{{$leader := sdict}}  
{{$leader.Set (str .User.ID) $damage}}  

{{dbSet 0 $leaderKey $leader}}

{{end}}

{{/* Result Embed */}}

{{$message := ""}}

{{if $atkBoost}}
{{$message = print $message "⚔️ Attack Boost Active (+25% Damage)\n"}}
{{end}}

{{if $defBoost}}
{{$message = print $message "🛡️ Defense Boost Active (-25% Damage Taken)\n"}}
{{end}}
{{if $tokenBoost}}
{{$message = print $message "🪙 Token Boost Active (x2 Boss Tokens)\n"}}
{{end}}
{{if $xpBoost}}
{{$message = print $message "✨ XP Boost Active (+10 XP)\n"}}
{{end}}

{{if $critical}}
{{$message = print $message "🔥 CRITICAL HIT!"}}
{{end}}

{{/* Combat Result Text */}}

{{$counterTitle := print $boss.emoji " " $boss.name " " $bossAction}}

{{if $bossCritical}}
{{$counterTitle = print "💥 " $boss.name " unleashes a DEVASTATING CRITICAL HIT!"}}
{{end}}

{{$counterMessage := print
"\n\n"
$counterTitle
"\n"
"💔 Damage Received: "
$bossDamage
"\n"
"❤️ Your HP: "
$profile.hp
"/"
$profile.maxHP
""
}}

{{$counterMessage = print
$counterMessage
$deathMessage
}}

{{$rewardText := print "🪙 Reward: **+" $tokenReward " Boss Token"}}

{{if ne $tokenReward 1}}
{{$rewardText = print $rewardText "s"}}
{{end}}

{{$rewardText = print $rewardText "**"}}

{{$xpText := printf "✨ XP Gained: **+%d XP**" $xpReward}}

{{if $xpBoost}}
    {{$xpText = printf "✨ XP Gained: **+%d XP** (Boost Applied)" $xpReward}}
{{end}}

{{$embed := cembed
"title" "⚔️ World Boss Attack"
"description" (printf
"<@%d> attacked %s %s\n\n💥 Damage: %d\n🪙 Reward: +%d Boss Token(s)\n\n%s\n%s\n\n❤️ Boss HP: %d / %d%s"
.User.ID
$boss.emoji
$boss.name
$damage
$tokenReward
$xpText
$message
$boss.hp
$boss.max_hp
$counterMessage
)
"color" $config.successColor
"footer" (sdict "text" $config.footer)
}}

{{sendMessage nil $embed}}

{{if $levelUp}}

{{sendMessage nil (cembed
"title" "🎉 LEVEL UP!"
"description" (printf
"Congratulations <@%d>!\n\n⭐ You reached Level %d!\n\n❤️ Max HP: +50\n⚔️ Attack: +2\n🛡️ Defense: +1\n\n💚 Your HP has been fully restored!"
.User.ID
$profile.level
)
"color" $config.successColor
"footer" (sdict "text" $config.footer)
)
}}

{{end}}