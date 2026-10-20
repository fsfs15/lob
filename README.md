# 🐺 Loup-Garou: Werewolf for Discord

A Discord bot for hosting Loup-Garou games with private role/action DMs, an interactive lobby, automatic day/night phases, and a SQLite leaderboard.

## Requirements

- Python 3.11+
- A Discord application/bot with the `bot` and `applications.commands` scopes
- The bot must be invited with permission to send messages and embeds in the game channel. Players need to allow DMs from server members.

## Run locally

1. Create and activate a virtual environment.
2. Install the project: `python -m pip install -e ".[test]"`
3. Copy `.env.example` to `.env` and set `DISCORD_TOKEN`.
4. Start the bot with `python -m loupgarou.bot`.

The bot syncs its slash commands when it connects. The SQLite database is created in the working directory by default; set `WEREWOLF_DATABASE` to choose another path.

## Deploy to Render

This bot runs continuously without receiving web requests, so deploy it as a **Background Worker**, not a Web Service. The included [`render.yaml`](./render.yaml) defines the worker, installs the project, starts the bot, and mounts a persistent disk at `/var/data` for the SQLite leaderboard.

1. Push this project to a GitHub repository.
2. In the [Render Dashboard](https://dashboard.render.com), choose **New > Blueprint** and connect the repository.
3. Review the `loup-garou-discord` worker and its persistent disk. Render background workers and persistent disks require a paid plan; check the current pricing in the dashboard before deploying.
4. When prompted, provide `DISCORD_TOKEN` as a secret. Do not put the token in `render.yaml` or commit it to the repository.
5. Deploy the Blueprint. Check the worker's logs for `Logged in as ...`; once connected, it syncs the slash commands with Discord.

The database is stored at `/var/data/werewolf.sqlite3` on the persistent disk. Keep the worker at one instance: local SQLite and a Render disk are not suitable for multiple instances. The disk preserves leaderboard data across deploys and restarts, but active game state is still in memory and a restart will end any running game.

## Commands

- `/werewolf start` — open a lobby (6–20 players; host joins automatically). Optional settings: `max_players`, `phase_seconds`, and `language` (`English` or `Français`).
- Click **Join** in the lobby, then the host uses `/werewolf launch`.
- `/werewolf status` — show the current phase and living players.
- `/werewolf cancel` — cancel an open lobby as its host.
- `/werewolf leaderboard` — show the top players by wins.
- `/werewolf stats` — show your personal wins and games played.

## Game roles and flow

The role deck scales with the player count and includes Werewolves, Villagers, Seer, Witch, Cupid, Hunter, and (from 10 players) Little Girl. The Seer investigates privately, the Witch has one save and one poison potion per game, Cupid chooses lovers on the first night, and lovers die together. Each night, wolves select a victim; each day, players vote privately. Voting ties cause no elimination. Villagers win when all wolves are eliminated; wolves win when they equal or outnumber the other living players.

Game messages support English and French. Phases time out using the configured `phase_seconds`. Players who have disabled DMs cannot receive actions; enable server-member DMs before launching. Active games are held in memory and do not survive a bot restart.

## Tests

Run `python -m pytest`.
