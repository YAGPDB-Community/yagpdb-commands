{{$config := sdict
    "profilePrefix" "rpg_profile_"
    "inventoryPrefix" "rpg_inventory_"

    "successColor" 3066993
    "errorColor" 15158332
    "embedColor" 3447003

    "footer" "World Boss System • Heal v1.0.0"
}}

{{/* =======================================================
   Item Database
======================================================= */}}

{{$items := sdict}}

{{$items.Set "hp" (sdict
    "effectType" "heal"
    "effectValue" 250
    "priority" 1
)}}

{{$items.Set "ghp" (sdict
    "effectType" "heal"
    "effectValue" 500
    "priority" 2
)}}

{{$items.Set "maxhp" (sdict
    "effectType" "fullHeal"
    "effectValue" 0
    "priority" 3
)}}

{{/* =======================================================
   Load Profile
======================================================= */}}

{{$profileKey := print $config.profilePrefix .User.ID}}

{{$profileDB := dbGet .User.ID $profileKey}}

{{if not $profileDB}}

{{sendMessage nil (cembed
"title" "❌ No RPG Profile"
"description" "Create your profile first using **!profile**."
"color" $config.errorColor
)}}

{{return}}

{{end}}

{{$profile := $profileDB.Value}}

{{/* =======================================================
   Load Inventory
======================================================= */}}

{{if not $profile.inventory}}
    {{$profile.Set "inventory" (sdict)}}
{{end}}

{{$itemsOwned := $profile.inventory}}

{{/* =======================================================
   Validation
======================================================= */}}

{{if ge $profile.hp $profile.maxHP}}

{{sendMessage nil (cembed
"title" "❤️ HP Full"
"description" "Your HP is already full."
"color" $config.errorColor
"footer" (sdict
"text" $config.footer
)
)}}

{{return}}

{{end}}

{{$missingHP := sub $profile.maxHP $profile.hp}}

{{$hpPercent := div (mult $profile.hp 100) $profile.maxHP}}

{{/* =======================================================
   Healing Logic
======================================================= */}}

{{$usedAlias := ""}}
{{$usedItem := sdict}}
{{$healAmount := 0}}

{{$hpCount := 0}}
{{$ghpCount := 0}}
{{$maxhpCount := 0}}

{{with (index $itemsOwned "hp")}}
    {{$hpCount = toInt .}}
{{end}}

{{with (index $itemsOwned "ghp")}}
    {{$ghpCount = toInt .}}
{{end}}

{{with (index $itemsOwned "maxhp")}}
    {{$maxhpCount = toInt .}}
{{end}}

{{/* Emergency Rule */}}

{{if and (le $hpPercent 10) (gt $maxhpCount 0)}}

    {{$usedAlias = "maxhp"}}
    {{$usedItem = index $items "maxhp"}}

{{else}}

    {{if and (le $missingHP 250) (gt $hpCount 0)}}

        {{$usedAlias = "hp"}}
        {{$usedItem = index $items "hp"}}

    {{else if and (le $missingHP 500) (gt $ghpCount 0)}}

        {{$usedAlias = "ghp"}}
        {{$usedItem = index $items "ghp"}}

    {{else if gt $ghpCount 0}}

        {{$usedAlias = "ghp"}}
        {{$usedItem = index $items "ghp"}}

    {{else if gt $hpCount 0}}

        {{$usedAlias = "hp"}}
        {{$usedItem = index $items "hp"}}

    {{else if gt $maxhpCount 0}}

        {{$usedAlias = "maxhp"}}
        {{$usedItem = index $items "maxhp"}}

    {{end}}

{{end}}

{{if eq $usedAlias ""}}

{{sendMessage nil (cembed
    "title" "❌ No Healing Potions"
    "description" "You don't have any healing potions."
    "color" $config.errorColor
    "footer" (sdict "text" $config.footer)
)}}

{{return}}

{{end}}

{{/* Apply Heal */}}

{{if eq $usedItem.effectType "fullHeal"}}

    {{$healAmount = $missingHP}}
    {{$profile.Set "hp" $profile.maxHP}}

{{else}}

    {{$healAmount = $usedItem.effectValue}}

    {{$newHP := add $profile.hp $healAmount}}

    {{if gt $newHP $profile.maxHP}}
        {{$newHP = $profile.maxHP}}
        {{$healAmount = $missingHP}}
    {{end}}

    {{$profile.Set "hp" $newHP}}

{{end}}

{{/* =======================================================
   Remove Used Potion
======================================================= */}}

{{$currentAmount := toInt (index $itemsOwned $usedAlias)}}

{{if le $currentAmount 1}}

    {{$itemsOwned.Del $usedAlias}}

{{else}}

    {{$itemsOwned.Set $usedAlias (sub $currentAmount 1)}}

{{end}}

{{/* =======================================================
   Save
======================================================= */}}

{{dbSet .User.ID $profileKey $profile}}


{{/* =======================================================
   Success Embed
======================================================= */}}

{{$itemName := ""}}
{{$itemEmoji := ""}}

{{if eq $usedAlias "hp"}}
    {{$itemName = "HP Potion"}}
    {{$itemEmoji = "❤️"}}
{{else if eq $usedAlias "ghp"}}
    {{$itemName = "Greater HP Potion"}}
    {{$itemEmoji = "💚"}}
{{else if eq $usedAlias "maxhp"}}
    {{$itemName = "Max HP Potion"}}
    {{$itemEmoji = "💖"}}
{{end}}

{{sendMessage nil (cembed
    "title" "❤️ Healing Successful"

    "description" (print
$itemEmoji " **" $itemName "** used successfully!\n\n"
"❤️ Restored HP: **" $healAmount "**\n\n"
"💚 Current HP: **" $profile.hp " / " $profile.maxHP "**"
)

    "color" $config.successColor

    "footer" (sdict
        "text" $config.footer
    )
)}}