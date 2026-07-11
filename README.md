![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Version](https://img.shields.io/badge/version-1.0.0-orange)
![License](https://img.shields.io/github/license/yerettegroup/squawk-bot)
![discord.py](https://img.shields.io/badge/discord.py-2.4%2B-5865F2)

# Squawk

A Discord bot that polls Yahoo Finance RSS feeds for a per-server stock ticker watchlist and general market news, posting new articles as `[Headline](<link>)` messages to configured channels.

## Setup

1. Create a Discord app at [discord.dev](https://discord.dev). No privileged intents required.
2. Invite the bot with the `bot` and `applications.commands` scopes, plus `Send Messages` permission.
3. Copy `.env.example` to `.env` and fill in `DISCORD_TOKEN`. `ALERT_USER_ID` is optional — set it to your Discord user ID to receive a DM when a feed starts failing.
4. Install dependencies: `pip install -r requirements.txt`
5. Run: `python bot.py`
6. In each server, use `/watchlist channel` and `/market channel` to configure where posts go.

## Commands

**Manage Server** members can use every command from any channel, cooldown-free. All setup and mutating commands (marked *admin* below) require Manage Server. The read-only commands are open to everyone, but you can restrict which channels they can be used in via `/config channel`.

| Command | Description | Cooldown |
|---|---|---|
| `/watchlist ticker action:<add\|remove> ticker:TICKER` | *admin* — Add/remove tickers (comma-separated, max 25). | 60s |
| `/watchlist channel action:<set\|clear>` | *admin* — Set the channel ticker news posts to. | 15s |
| `/watchlist show` | List tracked tickers. | 60s |
| `/ticker recent ticker:TICKER` | Fetch the 3 most recent articles for any ticker. | 30s |
| `/market channel action:<set\|clear>` | *admin* — Set the channel market news posts to. | 15s |
| `/market recent` | Fetch the 3 most recent market news articles. | 30s |
| `/config show` | *admin* — Show this server's full configuration (ephemeral). | — |
| `/config channel mode:<all\|none> exceptions:<#a #b …>` | *admin* — Set where regular users can use read-only commands. `all` = allowed everywhere; `none` = blocked everywhere. Exceptions flip the rule for the listed channels. Pass either param on its own, or both. | 15s |
| `/config blacklist action:<add\|remove\|show\|clear> pattern:text` | *admin* — Skip articles whose URL contains a given substring (e.g. `trefis`). | 15s |
| `/squawk` | Show bot version, uptime, tickers tracked, last poll, and any feeds in backoff. | 5s |

## How it works

Every 5 minutes, Squawk fetches the Yahoo Finance RSS feed for each tracked ticker and posts any unseen articles. Market news is pulled from major index feeds (S&P 500, Dow, Nasdaq, Russell 2000). Articles are deduplicated per-server by normalized URL (tracking params stripped). When a ticker is first added, existing articles are marked seen so there's no backlog dump.

Feed failures back off automatically after 3 consecutive errors and alert the server owner (and `ALERT_USER_ID` if set) via DM. A watchdog force-exits the process if the poll loop hangs, letting systemd restart it.

## Storage

All state lives in flat JSON files (no database), created automatically and gitignored:

| File | Contents |
|---|---|
| `watchlist.json` | Per-guild ticker lists |
| `seen.json` | Per-guild seen article URLs (normalized) |
| `config.json` | Per-guild ticker news channel |
| `news_config.json` | Per-guild market news channel |
| `permissions.json` | Per-guild read-only channel mode and exception list |
| `blacklist.json` | Per-guild URL substring blocklist |

## Running as a service

```
systemctl status squawk
systemctl restart squawk
journalctl -u squawk -f
```
