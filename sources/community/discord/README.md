# Discord Source

This source exposes Discord REST API data through Coral SQL using a Discord
bot token.

## Setup

1. Create an application in the Discord Developer Portal.
2. Add or reset the bot token on the Bot settings page.
3. Invite the bot to the guilds you want to query with the `bot` scope.
4. Grant the bot the permissions needed for your queries, such as View
   Channels and Read Message History.
5. Set `DISCORD_BOT_TOKEN` or pass it through `coral source add --interactive`.

```bash
DISCORD_BOT_TOKEN='...' coral source add --file ./sources/discord/manifest.yaml
coral source test discord
```

The `members` table may require enabling the GUILD_MEMBERS privileged intent
for the bot in the Developer Portal.

Treat bot tokens as secrets. If a token is pasted into an issue, pull request,
chat, or log, reset it in the Discord Developer Portal before using the bot in
production.

## Example Queries

```sql
SELECT id, name
FROM discord.guilds
ORDER BY name;
```

```sql
SELECT id, name, type
FROM discord.channels
WHERE guild_id = '123456789012345678'
ORDER BY position;
```

```sql
SELECT timestamp, author__username, content, permalink
FROM discord.messages
WHERE channel_id = '123456789012345678'
  AND guild_id = '123456789012345678'
LIMIT 50;
```

Discord's message and member APIs use cursor filters such as `before` and
`after` without returning a next cursor in the response body. Coral exposes
those as filters for manual paging.
