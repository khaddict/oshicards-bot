<p align="center">
  <img src="oshicards_logo.png" alt="oshicards logo" width="120">
</p>

<h1 align="center">oshicards-bot</h1>

<p align="center">
  <b>An auto pack opener bot for <a href="https://oshicards.com">oshicards</a>.</b><br>
  Opens your free boosters at a human pace and pings Discord when you pull something shiny.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/unofficial-fan%20tool-A855F7" alt="Unofficial fan tool">
  <img src="https://img.shields.io/badge/bash-4.2%2B-A855F7?logo=gnubash&logoColor=white" alt="Bash 4.2+">
  <img src="https://img.shields.io/badge/platform-linux-A855F7?logo=linux&logoColor=white" alt="Linux">
  <img src="https://img.shields.io/badge/license-MIT-A855F7" alt="MIT license">
</p>

`oshibot` is a single-file Bash tool: one script, no build step, and a guided setup.

```
09:14:02 pack 1
           common     Usopp (ONE PIECE)
           common     Hange Zoe (Attack on Titan) NEW
           common     Shoyo Hinata (HAIKYU!!)
           uncommon   Mob (Mob Psycho 100) NEW
           rare       Holo (Spice and Wolf)
09:14:21 pack 2 (golden)
           rare       Senku Ishigami (Dr. STONE) NEW
           rare       Maomao (The Apothecary Diaries)
           rare       Thorfinn (Vinland Saga) NEW
           epic       Frieren (Frieren: Beyond Journey's End)
           epic       Spike Spiegel (Cowboy Bebop) NEW
09:14:21 OK 2 pack(s) opened
```

> [!WARNING]
> 🚫 **Unofficial tool**, not affiliated with oshicards. Automating the game most likely breaks its terms of service and can get your account banned. Nothing here makes that risk go away. Use it at your own risk, ideally on an account you can afford to lose.

## ✨ Features

- 🎁 **Free boosters only.** Every request is sent with `maxGems: 0`, and the tool stops for good if a gem is ever spent.
- 🗓️ **A schedule of your own.** Each install draws its own random profile: bedtime, sleep length, session rhythm, weekday quiet hours, weekend shift. No two installs play alike.
- 🎮 **Four play styles.** `chill`, `regular`, `grinder` or `nonstop`, depending on how many packs you want and how much you want to stand out.
- 🪪 **Your browser's identity.** Setup copies your real user agent and language, so requests look like the browser you normally play with.
- 👀 **Looks like an open tab.** Loads the pack page before playing, keeps the site's device cookies, and sends the same presence heartbeat as the web app.
- 🛑 **Safety stop.** If the site answers anything unexpected (a block page, an unknown error), the tool stops and alerts you instead of insisting.
- 🔄 **Self-renewing session.** Paste your cookie once. The access token is refreshed automatically from then on.
- 📅 **Daily login streak.** Claims the daily login reward at the first session of the day, the same way the site's pages do when they load.
- 🔔 **Discord notifications.** Legendary and mythic pulls with the card art, a daily recap, and an alert if the tool stops.

## 📋 Requirements

Linux with `bash` 4.2+, `curl`, `jq` and GNU coreutils. `systemd` is only needed for the background service.

## 🚀 Quick start

```bash
git clone https://github.com/khaddict/oshicards-bot.git
cd oshicards-bot
sudo ./oshibot setup
```

The guided setup connects your account, asks for a play style, sets the Discord webhook and starts the service. After that, the `oshibot` command is available system-wide.

`sudo` is only needed to install the service. The service itself runs as the user who owns the folder, and your data files stay owned by that user.

Each step also exists on its own:

```bash
./oshibot cookie                 # connect your account
./oshibot profile grinder        # pick a play style
./oshibot webhook <discord-url>  # optional
./oshibot open                   # play one session now to check it works
sudo ./oshibot install           # start the background loop
```

## 🔑 Connecting your account

Run `./oshibot cookie` and follow the prompt. In short:

1. Open oshicards in a **private browsing window** of your usual browser and log in.
2. Press F12, open the **Console** tab and paste the line the tool shows you.
3. Paste the result into the prompt.
4. Close the private window **without logging out**.

The line copies your session cookie together with your browser's user agent and language. The private window matters: the tool takes over that session, so your regular browser stays logged in. Logging out would revoke the session the tool just saved.

You can also pass a file (`oshibot cookie paste.txt`) or pipe the paste on stdin.

## 🎮 Play styles

| Preset | Packs per day | Sessions per day | Looks like |
|---|---|---|---|
| 🌙 `chill` | 30 to 40 | 3 to 4 | a casual player |
| 🎯 `regular` | 50 to 70 | 4 to 6 | a dedicated player |
| 🔥 `grinder` | 100 to 110 | 10 to 12 | the realistic maximum for a human |
| 🤖 `nonstop` | 130 to 135 | 14 to 15 | nobody: it plays around the clock |

Figures come from two-week simulations of randomly drawn profiles. The theoretical maximum is 144 packs per day.

`nonstop` has no night break. It keeps the random gaps and pauses, but an account that plays every hour of every day is not a human one, so it is by far the easiest preset to spot. Only use it on an account you do not mind losing.

Free boosters regenerate at one every 10 minutes, up to a stock of 10. Some quests raise that limit (12 on the account used for testing). `oshibot` reads the real stock size and refill rate of your account from the site's answers, so nothing is hardcoded. Bonus boosters earned from quests are opened too, once the free stock is empty (set `USE_BONUS=0` in the profile to keep them). Emptying the stock just before it is full loses nothing, so even `grinder` never hammers the API.

Switch or redraw at any time with `oshibot profile <preset>`, and see what you got with `oshibot profile`:

```
Preset       : regular
Sessions     : every 104 to 187 min
Night        : around 23:20 to 07:05 (+/- 45 min, +75 min on weekends)
Quiet block  : 09:40 to 13:25 on weekdays
Between packs: 9 to 24 s
Light days   : 11% of days, partial sessions: 14%, delayed sessions: 9%
```

## 🎲 What a profile randomizes

| Behaviour | Drawn once per install | Varies every day |
|---|---|---|
| Bedtime | base between 22:30 and 02:30 | jitter, later on weekends |
| Sleep | 6 to 9 hours depending on the preset, none with `nonstop` | plus or minus 20 min |
| Session gap | its own minimum and maximum | each gap drawn in that range |
| Quiet block | start and length (work, school), weekdays only | end time |
| Pause between packs | its own range | each pause, with occasional long ones |
| Light days | how often | which days |
| Partial sessions | how often | which sessions leave part of the stock |

The profile is a plain `profile` file next to the script. You can edit it freely, and `git pull` never touches it. All times use the machine's local timezone.

## ⌨️ Commands

| Command | What it does |
|---|---|
| `oshibot setup` | Guided setup: account, play style, Discord, service |
| `oshibot cookie [file]` | Connect your account, with a step by step guide |
| `oshibot browser [file]` | Save your browser's identity without changing the session |
| `oshibot profile [preset]` | Show your profile, or draw a new one |
| `oshibot webhook <url>` | Set the Discord webhook (`test` to try it, `off` to remove it) |
| `oshibot open [n]` | Play one session now: empty the free stock, or `n` packs at most |
| `oshibot pause <duration>` | Stop playing for a while: `3d`, `12h`, `45m` |
| `oshibot resume` | Cancel a pause, or restart after a safety stop |
| `oshibot status` | Account, profile, next session, today's pulls |
| `oshibot stats` | Packs and rarities over the last 7 days |
| `oshibot install` / `uninstall` | Add or remove the systemd service |
| `oshibot logs` | Follow the service logs |

## 🛑 Safety stop

The tool halts itself, and tells you on Discord, when:

- the pack page or the open call answers `403` or `429`,
- the open call returns an error it does not know, or a response it cannot read,
- a pack is paid with gems instead of a free booster.

It then refuses to play until you have checked your account in a browser and run `oshibot resume`.

Two cases are handled differently:

- **Expired session.** Before stopping on an unknown answer, the tool checks whether its session is still valid. If not, it asks you to run `oshibot cookie` again instead.
- **Network and server errors** (`5xx`). The session is simply retried later.

## 📁 Files

Everything lives next to the script and is ignored by git:

| File | Content |
|---|---|
| `session.json` | Your auth tokens. Treat it like a password. |
| `device.json` | Browser identity, presence key and device cookies |
| `profile` | Your schedule and settings |
| `config` | Discord webhook URL |
| `history.tsv` | One line per opened pack, used by `stats` |
| `state`, `pause`, `halted` | Runtime state |

## ⚠️ Limitations

- **No guarantee against bans.** The schedule copies a human one, but requests still come from `curl` and from wherever you host the tool. A site that inspects TLS fingerprints or IP ranges can tell the difference. `CURL_BIN` in the profile lets you point to another curl build such as curl-impersonate; this is untested.
- **Volume is a signal on its own.** Even a perfect schedule stands out if it opens 100 packs every single day. Pick the lowest preset you are happy with.
- **This repository is public.** The site's admins can read it too.
- **Sessions can be revoked.** If the site ends the session, the service stops and tells you on Discord. Run `oshibot cookie` again. See the next section if it keeps happening.
- **Standard booster only.** The pack type is the `SLUG` setting.
- **It can break at any time.** The tool relies on private endpoints observed in the browser. A site update may change them.

## 🩹 The session keeps dying

You should only have to run `oshibot cookie` once. The access token lasts an hour and `oshibot` renews it by itself, for as long as the login behind it stays alive.

If the service stops after a few hours with `refresh_token_already_used`, a browser is using the **same login** as `oshibot`. Each renewal by one of them invalidates the other, and the site ends up logging both out.

The login you give to `oshibot` must belong to it alone:

1. Open a **new** private window and log in there. Do not reuse a tab that is already logged in.
2. Copy the cookie and paste it into `oshibot cookie`.
3. **Close that private window right away**, without logging out.

Your regular browser has its own login, so you can keep playing there. `oshibot cookie` warns you when the login you paste is more than 20 minutes old, which usually means it is shared with an open browser.

## ⚙️ How it works

The web app opens a pack with a single `POST /api/open` call, authenticated by a Supabase session cookie. The tool rebuilds that cookie from a stored session, renews the access token through the Supabase auth API when it is about to expire, and replays the same call with a fresh `requestId` each time.

The daily login reward is a Next.js server action that the site's pages call when they load. Its id changes with every site update, so `oshibot` reads it from the page's scripts and caches it until the site changes. Set `DAILY_LOGIN=0` in the profile to turn this off.

## 📄 License

[MIT](LICENSE)
