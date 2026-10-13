{{$config := sdict
    "profilePrefix" "rpg_profile_"

    "embedColor" 3447003
    "errorColor" 15158332

    "footer" "World Boss System • Shop v1.0.0"
}}

{{/* =======================================================
   Item Database
======================================================= */}}

{{$items := sdict}}

{{/* =======================================================
   ❤️ Healing
======================================================= */}}

{{$items.Set "hp" (sdict
    "id" "hp"
    "name" "HP Potion"
    "emoji" "❤️"

    "category" "healing"

    "buyPrice" 5
    "sellPrice" 2

    "stackable" true

    "effectType" "heal"
    "effectValue" 250

    "rarity" "common"

    "description" "Restores 250 HP."

    "enabled" true
)}}

{{$items.Set "ghp" (sdict
    "id" "ghp"
    "name" "Greater HP Potion"
    "emoji" "💚"

    "category" "healing"

    "buyPrice" 7
    "sellPrice" 5

    "stackable" true

    "effectType" "heal"
    "effectValue" 500

    "rarity" "uncommon"

    "description" "Restores 500 HP."

    "enabled" true
)}}

{{$items.Set "maxhp" (sdict
    "id" "maxhp"
    "name" "Max HP Potion"
    "emoji" "💖"

    "category" "healing"

    "buyPrice" 15
    "sellPrice" 9

    "stackable" true

    "effectType" "heal"
    "effectValue" -1

    "rarity" "rare"

    "description" "Fully restores your HP."

    "enabled" true
)}}

{{$items.Set "revive" (sdict
    "id" "revive"
    "name" "Revive Scroll"
    "emoji" "💫"

    "category" "special"

    "buyPrice" 100
    "sellPrice" 20

    "stackable" true

    "effectType" "revive"
    "effectValue" 1

    "rarity" "rare"

    "description" "Revives another fallen player. Used with !revive @User."

    "enabled" true
)}}

{{/* =======================================================
   Load Profile
======================================================= */}}

{{$profileKey := print $config.profilePrefix .User.ID}}

{{$profileDB := dbGet .User.ID $profileKey}}

{{if not $profileDB}}

    {{sendMessage nil (cembed
        "title" "❌ No RPG Profile"
        "description" "Create your profile first with **!profile**."
        "color" $config.errorColor
    )}}

    {{return}}

{{end}}

{{$profile := $profileDB.Value}}

{{$tokens := $profile.tokens}}

{{/* =======================================================
   Category
======================================================= */}}

{{$category := ""}}

{{if ge (len .CmdArgs) 1}}
    {{$category = lower (index .CmdArgs 0)}}
{{end}}

{{/* =======================================================
   Main Shop Menu
======================================================= */}}

{{if eq $category ""}}

{{$description := print
"🪙 **Your Boss Tokens:** **" $tokens "**\n\n"
"━━━━━━━━━━━━━━━━━━\n\n"
"❤️ **Consumables**\n"
"Healing Potions\n"
"Revive Scroll\n\n"
"⚡ **Boosts**\n"
"Temporary Buffs\n\n"
"🎒 **Inventory**\n"
"Backpack Upgrades\n\n"
"━━━━━━━━━━━━━━━━━━\n\n"
"**Commands**\n"
"`!shop consumables`\n"
"`!shop boosts`\n"
"`!shop inventory`\n"
}}

{{$embed := cembed
    "title" "🛒 WORLD BOSS SHOP"
    "description" $description
    "color" $config.embedColor
    "footer" (sdict
        "text" $config.footer
    )
}}

{{sendMessage nil $embed}}

{{return}}

{{end}}

{{/* =======================================================
   Consumables
======================================================= */}}

{{if eq $category "consumables"}}

{{$description := print
"🪙 **Your Boss Tokens:** **" $tokens "**\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"❤️ **HP Potion**\n"
"💰 **5 Boss Tokens**\n"
"Restores **25%** of your Max HP.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"💖 **Greater HP Potion**\n"
"💰 **7 Boss Tokens**\n"
"Restores **50%** of your Max HP.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"💚 **Max HP Potion**\n"
"💰 **15 Boss Tokens**\n"
"Fully restores your HP.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"💫 **Revive Scroll**\n"
"💰 **20 Boss Tokens**\n"
"Revives another fallen player.\n"
"Used with `!revive @User`.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"**Purchase Examples**\n"
"`!buy hp 5`\n"
"`!buy ghp 2`\n"
"`!buy maxhp 1`\n"
"`!buy revive 1`"
}}

{{$embed := cembed
    "title" "🛒 WORLD BOSS SHOP • ❤️ Consumables"
    "description" $description
    "color" $config.embedColor
    "footer" (sdict
        "text" $config.footer
    )
}}

{{sendMessage nil $embed}}
{{return}}

{{end}}

{{/* =======================================================
   ⚡ Boost Shop
======================================================= */}}

{{if eq $category "boosts"}}

{{$description := print
"🪙 **Your Boss Tokens:** **" $tokens "**\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"⚔️ **Attack Boost**\n"
"💰 **150 Boss Tokens**\n"
"+25% Attack for **1 hour**.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"🛡️ **Defense Boost**\n"
"💰 **150 Boss Tokens**\n"
"+25% Defense for **1 hour**.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"⭐ **XP Boost**\n"
"💰 **250 Boss Tokens**\n"
"+50% XP for **2 hours**.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"🪙 **Boss Token Boost**\n"
"💰 **300 Boss Tokens**\n"
"+50% Boss Tokens for **2 hours**.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"**Purchase Examples**\n"
"`!buy atk`\n"
"`!buy def`\n"
"`!buy xp`\n"
"`!buy token`"
}}

{{sendMessage nil (cembed
"title" "🛒 WORLD BOSS SHOP • ⚡ Boosts"
"description" $description
"color" $config.embedColor
"footer" (sdict
"text" $config.footer
)
)}}

{{return}}

{{end}}

{{/* =======================================================
   🎒 Inventory Shop
======================================================= */}}

{{if eq $category "inventory"}}

{{$description := print
"🪙 **Your Boss Tokens:** **" $tokens "**\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"🎒 **Inventory Expansion**\n"
"💰 **250 Boss Tokens**\n"
"+10 Inventory Slots.\n\n"

"━━━━━━━━━━━━━━━━━━\n\n"

"**Purchase Example**\n"
"`!buy bag`"
}}

{{sendMessage nil (cembed
    "title" "🛒 WORLD BOSS SHOP • 🎒 Inventory"
    "description" $description
    "color" $config.embedColor
    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}

{{end}}

{{/* =======================================================
   ⚔️ Boosts
======================================================= */}}

{{$items.Set "atk" (sdict
    "id" "atk"
    "name" "Attack Boost"
    "emoji" "⚔️"

    "category" "boost"

    "buyPrice" 150
    "sellPrice" 75

    "stackable" true

    "effectType" "attackBoost"
    "effectValue" 25

    "duration" 3600

    "rarity" "uncommon"

    "description" "Increase Attack by 25% for 1 hour."

    "enabled" true
)}}

{{$items.Set "def" (sdict
    "id" "def"
    "name" "Defense Boost"
    "emoji" "🛡️"

    "category" "boost"

    "buyPrice" 150
    "sellPrice" 75

    "stackable" true

    "effectType" "defenseBoost"
    "effectValue" 25

    "duration" 3600

    "rarity" "uncommon"

    "description" "Increase Defense by 25% for 1 hour."

    "enabled" true
)}}

{{$items.Set "xp" (sdict
    "id" "xp"
    "name" "XP Boost"
    "emoji" "⭐"

    "category" "boost"

    "buyPrice" 250
    "sellPrice" 125

    "stackable" true

    "effectType" "xpBoost"
    "effectValue" 50

    "duration" 7200

    "rarity" "rare"

    "description" "Gain 50% more XP for 2 hours."

    "enabled" true
)}}

{{$items.Set "token" (sdict
    "id" "token"
    "name" "Boss Token Boost"
    "emoji" "🪙"

    "category" "boost"

    "buyPrice" 300
    "sellPrice" 150

    "stackable" true

    "effectType" "tokenBoost"
    "effectValue" 50

    "duration" 7200

    "rarity" "rare"

    "description" "Gain 50% more Boss Tokens for 2 hours."

    "enabled" true
)}}

{{/* =======================================================
   🎒 Inventory
======================================================= */}}

{{$items.Set "bag" (sdict
    "id" "bag"
    "name" "Inventory Expansion"
    "emoji" "🎒"

    "category" "inventory"

    "buyPrice" 250
    "sellPrice" 125

    "stackable" false

    "effectType" "inventorySlots"
    "effectValue" 10

    "rarity" "rare"

    "description" "Increase your inventory capacity by 10 slots."

    "enabled" true
)}}


{{/* =======================================================
   Invalid Category
======================================================= */}}

{{sendMessage nil (cembed
    "title" "❌ Unknown Shop Category"

    "description" (print
"Please choose one of the available shop categories.\n\n"

"❤️ **Consumables**\n"
"`!shop consumables`\n\n"

"⚡ **Boosts**\n"
"`!shop boosts`\n\n"

"🎒 **Inventory**\n"
"`!shop inventory`\n\n"
    )

    "color" $config.errorColor

    "footer" (sdict
        "text" $config.footer
    )
)}}

{{return}}
