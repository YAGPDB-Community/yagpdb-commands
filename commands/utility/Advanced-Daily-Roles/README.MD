🎁 GameCentral Daily System

A fully automated Daily Role Reward System for Discord servers using YAGPDB Custom Commands.

Members can claim one Daily Role every 24 hours, with randomized rarities, Troll Events, Daily Streaks, a Pity System and special Weekend Bonuses.

Built for GameCentral with a modular and future-ready configuration.

🎮 Command

Command| Description "!daily"| Claim your Daily Role

🇬🇷 Timezone

The cooldown expiration display supports Greek local time.

Summer

+3 hours

Current configuration:

{{$next := ($cd.ExpiresAt.Add 10800e9).Format "02/01/2006 στις 15:04:05"}}

Winter

Change:

10800e9

to:

7200e9

This changes the display from UTC+3 to UTC+2.

«⚠️ The cooldown itself remains 24 hours. This offset only affects the displayed availability time.»
