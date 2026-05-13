# Discord Source

This source exposes Discord REST API data through Coral SQL using a Discord
bot token.

## Authentication

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications).
2. Add or reset the bot token on the Bot settings page.
3. Invite the bot to the guilds you want to query with the `bot` scope.
4. Grant the bot the permissions needed for your queries, such as **View
   Channels**, **Read Message History**, and any guild-specific permissions
   needed for the `members` or `roles` tables.
5. Set the `DISCORD_BOT_TOKEN` environment variable before running Coral:
   ```bash
   export DISCORD_BOT_TOKEN='your-bot-token-here'
   coral source add --file manifest.yaml
   coral source test discord
   ```

The `members` table requires the **GUILD_MEMBERS** privileged intent to be
enabled for the bot in the Developer Portal under **Bot** > **Privileged
Gateway Intents**.

The `messages` table can return empty `content`, `embeds`, `attachments`, and
other message-content fields unless the bot has the **MESSAGE_CONTENT**
privileged intent enabled or Discord grants one of its documented exceptions.

**Important:** Treat bot tokens as secrets. If a token is accidentally pasted
into an issue, pull request, chat, or log, reset it immediately in the Discord
Developer Portal before using the bot in production.

## Rate Limits

Discord enforces per-endpoint and per-bot rate limits. Coral respects the
`Retry-After` header in rate-limited responses and will automatically retry
requests.

Typical rate limits:
- Most endpoints: 50–100 requests per second
- `messages` endpoint: 5 requests per 5 seconds
- `members` endpoint: 1 request per 1 second for large guilds

Be cautious when querying `messages` or `members` for large guilds; consider
using cursor filters to page through results in smaller batches.

## Schema Overview

**Start here:**
- `current_user`: Confirm bot identity and token validity
- `guilds`: Discover available guilds and their IDs

**Guild-scoped tables** (require `guild_id`):
- `channels`: List channels in a guild
- `members`: List members in a guild (requires GUILD_MEMBERS intent)
- `roles`: List roles in a guild

**Channel-scoped tables** (require `channel_id`):
- `messages`: Query recent messages in a channel

## Pagination and Cursor Filters

The `messages` and `members` tables use cursor-based pagination. Discord does
not include a next-cursor in either response, so pagination uses IDs from the
rows already returned.

For `messages`, Discord returns results in reverse chronological order (newest
first). Use optional `before`, `after`, or `around` filters with Discord message
IDs. To fetch older pages from a newest-first batch, pass the oldest message ID
from the previous result as `before` in your next query.

For `members`, use the optional `after` filter with a Discord user ID cursor to
request members after that user ID.

Example:
```sql
-- First batch of messages
SELECT id, timestamp, author__username, content
FROM discord.messages
WHERE channel_id = '123456789012345678'
LIMIT 50;

-- Next batch: use the oldest id from the previous result
SELECT id, timestamp, author__username, content
FROM discord.messages
WHERE channel_id = '123456789012345678'
  AND before = '999888777666555444'
LIMIT 50;
```

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
