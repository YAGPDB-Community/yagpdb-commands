{{$config := sdict
    "profilePrefix" "rpg_profile_"
    "successColor" 3066993
    "errorColor" 15158332
    "reviveCooldown" 3600
    "footer" "World Boss System v2.0.0"
}}

{{$isAdmin := hasPermissions 8}}

{{/* =======================================================
   Load Reviver Profile
======================================================= */}}

{{$reviverKey := print $config.profilePrefix .User.ID}}
{{$reviverDB := dbGet .User.ID $reviverKey}}

{{if not $reviverDB}}

    {{sendMessage nil (cembed
        "title" "❌ No RPG Profile"
        "description" "Create your profile first with `!profile`."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{$reviver := $reviverDB.Value}}

{{/* =======================================================
   NO TARGET → ADMIN SELF REVIVE ONLY
======================================================= */}}

{{if not .CmdArgs}}

    {{if not $isAdmin}}

        {{sendMessage nil (cembed
            "title" "🔒 Admin Only"
            "description" "Only administrators can use `!revive` without a target.\n\nTo revive another player, use:\n`!revive @User`"
            "color" $config.errorColor
            "footer" (sdict "text" $config.footer)
        )}}

        {{return}}

    {{end}}

    {{if $reviver.alive}}

        {{sendMessage nil (cembed
            "title" "❤️ Already Alive"
            "description" "You are already ready for battle!"
            "color" $config.errorColor
            "footer" (sdict "text" $config.footer)
        )}}

        {{return}}

    {{end}}

    {{$reviver.Set "alive" true}}
    {{$reviver.Set "hp" $reviver.maxHP}}

    {{dbSet .User.ID $reviverKey $reviver}}

    {{sendMessage nil (cembed
        "title" "✨ Resurrection Complete!"
        "description" (printf
            "<@%d> has been resurrected!\n\n❤️ HP Restored: **%d / %d**\n\n⚔️ You can attack the World Boss again!"
            .User.ID
            $reviver.hp
            $reviver.maxHP
        )
        "color" $config.successColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   TARGET REVIVE
======================================================= */}}

{{$target := userArg (index .CmdArgs 0)}}

{{if eq $target.ID .User.ID}}

    {{sendMessage nil (cembed
        "title" "❌ Invalid Target"
        "description" "To revive yourself, use `!revive`."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* Load Target Profile */}}

{{$targetKey := print $config.profilePrefix $target.ID}}
{{$targetDB := dbGet $target.ID $targetKey}}

{{if not $targetDB}}

    {{sendMessage nil (cembed
        "title" "❌ No RPG Profile"
        "description" (printf
            "<@%d> does not have an RPG profile."
            $target.ID
        )
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{$targetProfile := $targetDB.Value}}

{{/* Target must be dead */}}

{{if $targetProfile.alive}}

    {{sendMessage nil (cembed
        "title" "❤️ Already Alive"
        "description" (printf
            "<@%d> is already alive and ready for battle!"
            $target.ID
        )
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   ADMIN TARGET REVIVE
======================================================= */}}

{{if $isAdmin}}

    {{$targetProfile.Set "alive" true}}
    {{$targetProfile.Set "hp" $targetProfile.maxHP}}

    {{dbSet $target.ID $targetKey $targetProfile}}

    {{sendMessage nil (cembed
        "title" "✨ Resurrection Complete!"
        "description" (printf
            "<@%d> has been resurrected by <@%d>!\n\n❤️ HP Restored: **%d / %d**"
            $target.ID
            .User.ID
            $targetProfile.hp
            $targetProfile.maxHP
        )
        "color" $config.successColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   PLAYER COOLDOWN
======================================================= */}}

{{$cooldownKey := print "revive_cd_" .User.ID}}
{{$cooldownDB := dbGet 0 $cooldownKey}}

{{if $cooldownDB}}

    {{$expires := $cooldownDB.ExpiresAt.Unix}}

    {{sendMessage nil (cembed
        "title" "⏳ REVIVE COOLDOWN"
        "description" (printf
            "💫 You recently revived another player.\n\nYou can use another Revive Scroll **<t:%d:R>**."
            $expires
        )
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   Check Revive Scroll
======================================================= */}}

{{if not $reviver.inventory}}
    {{$reviver.Set "inventory" (sdict)}}
{{end}}

{{$inventory := $reviver.inventory}}
{{$scrolls := 0}}

{{with index $inventory "revive"}}
    {{$scrolls = toInt .}}
{{end}}

{{if le $scrolls 0}}

    {{sendMessage nil (cembed
        "title" "❌ No Revive Scroll"
        "description" "You don't have a **💫 Revive Scroll**.\n\nPurchase one from the Shop to revive another player."
        "color" $config.errorColor
        "footer" (sdict "text" $config.footer)
    )}}

    {{return}}

{{end}}

{{/* =======================================================
   Consume Scroll + Revive
======================================================= */}}

{{if eq $scrolls 1}}
    {{$inventory.Del "revive"}}
{{else}}
    {{$inventory.Set "revive" (sub $scrolls 1)}}
{{end}}

{{$targetProfile.Set "alive" true}}
{{$targetProfile.Set "hp" $targetProfile.maxHP}}

{{$reviver.Set "inventory" $inventory}}

{{/* Save both profiles */}}

{{dbSet .User.ID $reviverKey $reviver}}
{{dbSet $target.ID $targetKey $targetProfile}}

{{/* Start cooldown AFTER successful revive */}}

{{dbSetExpire 0 $cooldownKey true $config.reviveCooldown}}

{{/* =======================================================
   Result
======================================================= */}}

{{sendMessage nil (cembed
    "title" "💫 Resurrection Complete!"
    "description" (printf
        "<@%d> used a **💫 Revive Scroll** to resurrect <@%d>!\n\n❤️ HP Restored: **%d / %d**\n\n💫 **1 Revive Scroll consumed.**"
        .User.ID
        $target.ID
        $targetProfile.hp
        $targetProfile.maxHP
    )
    "color" $config.successColor
    "footer" (sdict "text" $config.footer)
)}}