# mancavebot

A community Discord bot with sass, scams detection, and actually useful features. Runs on the [discordbotcluster](https://github.com/anomalyco/discordbotcluster) platform.

## Features

### Commands

| Command | Description |
|---|---|
| `/suggest <suggestion>` | Posts a suggestion embed with upvote/downvote reactions in the current channel |
| `/remindme <duration> <message>` | Sets a reminder — bot DMs you when time's up (e.g. `30m`, `1h`, `2h30m`, `1d`) |
| `/roast [user]` | Drops a random roast. No target = self-roast |
| `/wyr` | Generates a random "Would You Rather…" question |
| `/rate <thing>` | Rates anything 0–10 with a sarcastic comment |
| `/vibecheck [user]` | Checks someone's vibe with a random result |

### Anti-Scam Module

Automatically scans all messages for:

- **External Discord invites** — resolves invite codes via Discord API; deletes if the invite points to a different server
- **Invalid invite links** — catches broken invite URLs
- **Common scam patterns** — fake Nitro giveaways, Steam gift card scams, "you won" phishing, fake staff/partner programs

When flagged: message is deleted, user receives a DM explaining why, and optionally logs to a configured channel.

### Background Services

- **Reminder checker** — checks every 30s for due reminders and delivers them via DM
- **Presence rotation** — cycles through funny status messages every 45s

## Setup

### Prerequisites

- Node.js 20+
- A Discord bot application with a token from the [Discord Developer Portal](https://discord.com/developers/applications)

### Discord Intents

Enable these in the Developer Portal under **Bot → Privileged Gateway Intents**:

- `MESSAGE CONTENT INTENT` — required for the anti-scam module to read message content

### Installation

```bash
cd bots/mancavebot
npm install
```

### Configuration

The bot is managed by the parent cluster. Add it to `config/bots.json`:

```json
{
  "name": "mancavebot",
  "main": "index.js",
  "token": "<encrypted token>",
  "enabled": true
}
```

### Database

The bot uses [sql.js](https://github.com/sql-js/sql.js) (pure JavaScript SQLite — no native compilation needed). A `data.db` file is created automatically on first run.

#### Config Table

Set these via the parent cluster's interactive console or directly in the DB:

| Key | Value |
|---|---|
| `antiscam_enabled` | `"true"` or `"false"` (defaults to enabled) |
| `antiscam_log_channel` | Channel ID for moderation logs |

## Project Structure

```
mancavebot/
├── index.js              # Entry point — client setup, command loader, presence, reminder checker
├── db.js                 # SQLite wrapper (sql.js) with parameterized queries
├── package.json
├── commands/
│   ├── suggest.js        # /suggest
│   ├── remindme.js       # /remindme
│   ├── roast.js          # /roast
│   ├── wyr.js            # /wyr
│   ├── rate.js           # /rate
│   └── vibecheck.js      # /vibecheck
├── modules/
│   └── antiscam.js       # Anti-scam message scanner
└── data.db               # SQLite database (auto-created)
```

## Adding a Command

1. Create a new file in `commands/` that exports `{ data, execute }`.
2. Use `SlashCommandBuilder` to define the command schema.
3. The command handler auto-loads it on next start.

## Timezone

The bot operates in `Europe/Helsinki` timezone — set at the top of `index.js`.

## Development

```bash
node index.js
```

Runs standalone (without the cluster). Note: the cluster wrapper sends the token via stdin, so standalone mode requires the token to be injected externally.
