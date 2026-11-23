<p align="center">
  <img src="oshicards_logo.png" alt="oshicards logo" width="120">
</p>

<h1 align="center">oshicards-bot</h1>

<p align="center">
  <b>An auto pack opener bot for <a href="https://oshicards.com">oshicards</a>.</b><br>
  Opens your free boosters at a human pace, plays the daily mini-games, does your missions and pings Discord when you pull something shiny.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/unofficial-fan%20tool-A855F7" alt="Unofficial fan tool">
  <img src="https://img.shields.io/badge/bash-4.2%2B-A855F7?logo=gnubash&logoColor=white" alt="Bash 4.2+">
  <img src="https://img.shields.io/badge/platform-linux-A855F7?logo=linux&logoColor=white" alt="Linux">
  <img src="https://img.shields.io/badge/license-MIT-A855F7" alt="MIT license">
</p>

`oshibot` is a single-file Bash tool: one script, no build step, and a guided setup.

```
09:14:02 pack 1 (rivals-2026)
           common     Usopp (ONE PIECE)
           common     Hange Zoe (Attack on Titan) NEW
           common     Shoyo Hinata (HAIKYU!!)
           uncommon   Mob (Mob Psycho 100) NEW
           rare       Holo (Spice and Wolf)
09:14:21 pack 2 (rivals-2026) (golden)
           rare       Senku Ishigami (Dr. STONE) NEW
           rare       Maomao (The Apothecary Diaries)
           rare       Thorfinn (Vinland Saga) NEW
           epic       Frieren (Frieren: Beyond Journey's End)
           mythic     Spike Spiegel [Rivals] (Cowboy Bebop) NEW
09:14:21 OK 2 pack(s) opened
```

> [!WARNING]
> 🚫 **Unofficial tool**, not affiliated with oshicards. The site's [rules](https://oshicards.com/en/rules) forbid scripts and bots: a suspicious account can be frozen, and serious abuse leads to a ban. Nothing here makes that risk go away. Use it at your own risk, ideally on an account you can afford to lose.

## ✨ Features

- 🎁 **Free boosters only.** Every pack is opened with `maxGems: 0`, and the tool stops itself if a gem is ever spent.
- 📦 **Any free pack.** Standard by default. Event and themed packs can be listed in order of preference, with an automatic fallback when an event ends.
- 🗓️ **A schedule of your own.** Each install draws its own random profile: bedtime, sleep length, session rhythm, weekday quiet hours, weekend shift.
- 🎮 **Four play styles.** `chill`, `regular`, `grinder` or `nonstop`, depending on how many packs you want and how much you want to stand out.
- 🪪 **Looks like your browser.** Copies your user agent and language, loads the pages before acting, keeps the site's device cookies and sends the same presence heartbeat as the web app.
- 🔄 **Self-renewing session.** Paste your cookie once. The access token is renewed automatically from then on.
- 🌙 **Nightly routine (opt-in).** Between 00:01 and 01:00, plays four mini-games, does and claims the daily and weekly missions, claims the other rewards, then posts a recap.
- 🔔 **Quiet Discord.** Rare pulls with the card art, alerts when the tool stops or needs you, and one recap a night. Nothing else.
- 🛑 **Safety stop.** If the site answers anything unexpected, the tool stops and alerts you instead of insisting.

## 📋 Requirements

Linux with `bash` 4.2+, `curl`, `jq`, GNU coreutils, GNU `grep` and `flock` (util-linux). All of them ship with common distributions, except sometimes `jq`. The "Who is it?" quiz also needs Python 3 with Pillow (`apt install python3-pil`). `systemd` is only needed for the background service.

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

The mini-games, missions and rewards are off until you turn them on in the `profile` file (see [Nightly routine](#-nightly-routine)).

## 🔑 Connecting your account

Run `./oshibot cookie` and follow the prompt. In short:

1. Open oshicards in a **new private browsing window** of your usual browser and log in.
2. Press F12, open the **Console** tab and paste the line the tool shows you.
3. Paste the result into the prompt.
4. Close the private window **right away, without logging out**.

The line copies your session cookie together with your browser's user agent and language. The private window matters: the tool takes over that login, and your regular browser keeps its own. Logging out would revoke the session the tool just saved, and leaving the window open makes the two log each other out (see [The session keeps dying](#-the-session-keeps-dying)).

You can also pass a file (`oshibot cookie paste.txt`) or pipe the paste on stdin.

## 🎮 Play styles

| Preset | Packs per day | Sessions per day | Looks like |
|---|---|---|---|
| 🌙 `chill` | 30 to 40 | 3 to 4 | a casual player |
| 🎯 `regular` | 50 to 70 | 4 to 6 | a dedicated player |
| 🔥 `grinder` | 95 to 110 | 10 to 12 | the realistic maximum for a human |
| 🤖 `nonstop` | 125 to 135 | 14 to 15 | nobody: it plays around the clock |

Figures come from simulations of randomly drawn profiles over one to two weeks, with a stock of 12 boosters. The theoretical maximum is 144 packs per day.

`nonstop` has no night break. It keeps the random gaps and pauses, but an account that plays every hour of every day is not a human one, so it is by far the easiest preset to spot.

Free boosters regenerate at one every 10 minutes, up to a stock of 10 that the season pass and some quests raise. `oshibot` reads the real stock size and refill rate of your account from the site's answers. Bonus boosters are opened too, once the free stock is empty (`USE_BONUS=0` keeps them). Emptying the stock just before it is full loses nothing, so even `grinder` never hammers the site.

Switch or redraw at any time with `oshibot profile <preset>`, and see what you got with `oshibot profile`:

```
Preset       : regular
Sessions     : every 104 to 187 min
Night        : around 23:20 to 07:05 (+/- 45 min, +75 min on weekends)
Quiet block  : 09:40 to 13:25 on weekdays
Between packs: 9 to 24 s
Light days   : 11% of days, partial sessions: 14%, delayed sessions: 9%
```

### What a profile randomizes

| Behaviour | Drawn once per install | Varies every day |
|---|---|---|
| Bedtime | base between 22:30 and 02:30 | jitter, later on weekends |
| Sleep | 6 to 9 hours depending on the preset, none with `nonstop` | plus or minus 20 min |
| Session gap | its own minimum and maximum | each gap drawn in that range |
| Quiet block | start and length (work, school), weekdays only, `chill` and `regular` | end time |
| Pause between packs | its own range | each pause, with occasional long ones |
| Light days | how often (none with `nonstop`) | which days |
| Partial sessions | how often | which sessions leave part of the stock |

## 📦 Choosing the packs

All free packs draw from the same free stock. By default `oshibot` opens Standard packs. To open others, list them in the `profile` file, in order of preference:

```bash
PACKS="rivals-2026 standard"
```

At each session, `oshibot` reads the pack list from the site and opens the first one that is free and available right now. Here, Rivals packs while the event lasts, then Standard packs once it ends, with nothing to change. A pack that costs gems is skipped with a warning, even if you list it.

`oshibot packs` shows every pack the site offers, with its cost and availability window:

```
Pack                   Name                       Cost   Available
standard               Standard                   free   always <- in your PACKS
shonen                 Shōnen                     free   always
weekly                 Weekly banner              gems   always
rivals-2026            Rivals                     free   until Sat 28 Nov 19:00 <- in your PACKS
collector-prestige     Prestige Edition           gems   always
```

`oshibot open 3 shonen` opens three packs of a given type right away, whatever `PACKS` says.

## 🌙 Nightly routine

Every night, at a random time between `ROUTINE_FROM` and `ROUTINE_TO` (00:01 and 01:00 by default, Paris time, like the site's day), `oshibot`:

1. plays the rewarded game of each mini-game in `PLAY_GAMES`,
2. does the missions in `DO_MISSIONS` and claims every finished mission,
3. claims the other rewards: weekly boss, season pass tiers, achievements, completed anime,
4. posts the recap on Discord.

It runs even during the night break of your preset. `oshibot routine` runs it right away. During the day, missions are still checked and claimed every two to four hours, after a pack session.

```bash
PLAY_GAMES="anime rarer memory quiz"
DO_MISSIONS="banner wishlist chat auction sell exchange recycle"
MISSION_FRIEND="eikosensei"   # pick one with: oshibot friend
AUCTION_MAX_GEMS=50
CLAIM_MISSIONS=1              # also claim the other rewards; finished missions are always claimed
```

### Mini-games

| Game | How it is played | Reward |
|---|---|---|
| Mystery anime (`anime`) | The site shows a character. Its card page on the site names its anime, which is the answer. | up to 30 gems |
| Rarer? (`rarer`) | Uses the rarity of cards already seen, else the character's popularity on AniList. One wrong answer ends the game. | 3 gems per correct answer |
| Memory pairs (`memory`) | Nothing is revealed before a card is flipped, so it remembers every card seen and never flips a known pair wrong. | 25 gems in 8 moves or fewer, 15 up to 11, 10 beyond |
| Who is it? (`quiz`) | Two throwaway guesses reveal the anime. The pixelated picture is then compared with the pictures of that anime's cards. | up to 50 gems, on top of the daily cap |

The site caps the mini-games at 50 gems a day in total, "Who is it?" excepted. "Scrambled page", "Daily dungeon" and the multiplayer games are not played.

**About "Who is it?".** Its reward grows with your daily streak (5 gems a day, up to 50), and a missed day resets the streak. So the last guess is only sent when one card clearly stands out. Otherwise you get the anime and the two best candidates on Discord, and you finish it yourself. Wrong guesses do not lower the reward.

**About "Rarer?".** Every card from an opened pack, and every card revealed by a game, is saved with its true rarity in `rarity.tsv`. When both cards of a round are known, the answer is certain. Otherwise rarity follows popularity: the card's picture address holds its AniList character id, and `oshibot` asks the public [AniList API](https://anilist.co) for the favourites of both characters. This is the only request sent to a third party, and it only contains the two ids. A few classics are always Mythic on the site whatever their popularity; they are learned the first time they come up.

### Missions

| Mission | What the tool does |
|---|---|
| Open the weekly banner pack (`banner`) | Opens the banner's free pack of the day, never a paid one. |
| Add a card to your wishlist (`wishlist`) | Wishes a card you do not own yet. |
| Send a message to a friend (`chat`) | Sends a short friendly message to `MISSION_FRIEND`, and only to them. |
| Bid on an auction (`auction`) | Places the minimum bid on the cheapest auction, up to `AUCTION_MAX_GEMS`. You may win it and pay. |
| Sell a card on the market (`sell`) | Lists one duplicate for 48 hours at the site's suggested price, then waits for a buyer. |
| Exchange duplicates (`exchange`) | Gives 5 common duplicates for 1 random uncommon. |
| Recycle duplicates (`recycle`) | Recycles as many duplicates as the mission asks, for gems. |

`sell`, `exchange` and `recycle` only touch common and uncommon duplicates numbered above 100. The site already keeps your best copy of each card, your locked copies and copies on sale out of reach. Missions that progress on their own (pulls, new cards, mini-games, quiz) need nothing more.

### Missions to check

At the end of the routine, `oshibot` looks at the missions still open. Those it cannot finish, a mission it does not know or one that failed, are posted to Discord in one "Missions to check" message. Weekly missions that progress on their own, and a card still waiting for a buyer, are reported on their last day if still open.

**No message means every mission is done or on its way.**

### Rewards

With `CLAIM_MISSIONS=1`, the routine also visits the games, pass, achievements and "my anime" pages, and claims what is waiting there: the weekly boss reward (100 gems per defeated boss), season pass tiers including the bonus ones after tier 30, unlocked achievements and completed anime.

Games played without mistakes every day, and rewards claimed within hours, are a stronger automation signal than opening packs. Leave all of this off if you want to stay discreet.

## 🔔 Discord

Set a webhook with `oshibot webhook <url>` (Discord > channel > Edit > Integrations > Webhooks > New Webhook > Copy Webhook URL), and try it with `oshibot webhook test`.

| Message | When |
|---|---|
| A card with its art | as soon as it is pulled, for the rarities in `NOTIFY_RARITIES` (legendary and mythic by default) |
| 🌙 Daily recap | after the nightly routine: yesterday's packs and rarities, mythic pulls, mini-game results, missions done and claimed, rewards, login streak |
| ⚠️ Missions to check | after the routine, only when a mission needs you |
| "Who is it?" needs you | when the quiz picture matches no card clearly enough |
| Stopped, session lost | when the tool stops itself |

`oshibot recap preview` prints the recap without posting it. `DAILY_RECAP=0` turns it off.

## ⌨️ Commands

| Command | What it does |
|---|---|
| `oshibot setup` | Guided setup: account, play style, Discord, service |
| `oshibot cookie [file]` | Connect your account, with a step by step guide |
| `oshibot browser [file]` | Save your browser's identity without changing the session |
| `oshibot profile [preset]` | Show your profile, or draw a new one, keeping your settings |
| `oshibot webhook <url>` | Set the Discord webhook (`test` to try it, `off` to remove it) |
| `oshibot open [n] [pack]` | Play one session now: empty the free stock, or `n` packs at most |
| `oshibot packs` | List the site's packs, their cost and availability, and which ones you open |
| `oshibot games [name...] [free]` | Play today's mini-games now (`free` replays without reward) |
| `oshibot missions` | Do and claim the missions now |
| `oshibot routine` | Run the nightly routine now: games, missions, rewards, recap |
| `oshibot recap [preview]` | Post the recap now, or print it without posting |
| `oshibot friend [username]` | List your friends, or pick the one for the message mission |
| `oshibot pause <duration>` | Stop playing for a while: `3d`, `12h`, `45m` |
| `oshibot resume` | Cancel a pause, or restart after a safety stop |
| `oshibot status` | Account, profile, stock, games, routine, next session, today's pulls |
| `oshibot stats` | Packs and rarities over the last 7 days |
| `oshibot install` / `uninstall` | Add or remove the systemd service |
| `oshibot logs` | Follow the service logs |
| `oshibot run` | Run the loop in the foreground, without systemd |

## ⚙️ Settings

Everything lives in the `profile` file next to the script. It is plain Bash variables, you can edit it freely, and `git pull` never touches it. Run `oshibot profile <preset>` to redraw the schedule: your other settings are kept.

| Setting | Default | What it does |
|---|---|---|
| `PACKS` | `standard` | Packs to open, in order of preference |
| `USE_BONUS` | `1` | Also open bonus boosters once the free stock is empty |
| `DAILY_LOGIN` | `1` | Claim the daily login reward |
| `PLAY_GAMES` | empty | Mini-games of the routine: `anime rarer memory quiz` |
| `DO_MISSIONS` | empty | Missions to do: `banner wishlist chat auction sell exchange recycle` |
| `CLAIM_MISSIONS` | `0` | Claim finished missions and the other rewards (missions alone are also claimed when `DO_MISSIONS` is set) |
| `MISSION_FRIEND` | empty | Who gets the message of the friend mission |
| `AUCTION_MAX_GEMS` | `50` | The most gems a mission bid may engage |
| `ROUTINE_FROM`, `ROUTINE_TO` | `00:01`, `01:00` | Window of the nightly routine, Paris time |
| `NOTIFY_RARITIES` | `["legendary","mythic"]` | Rarities posted to Discord as they are pulled |
| `DAILY_RECAP` | `1` | Post the nightly recap |
| `CURL_BIN` | `curl` | Another curl build, such as curl-impersonate (untested) |

The schedule settings above them (gaps, night, pauses) are drawn by the preset and commented in the file. Pack session times use the machine's timezone; the routine and the mini-games follow Paris time.

## 🛑 Safety stop

The tool halts itself, and tells you on Discord, when:

- a page of the site, the open call or an action answers `403` or `429`,
- the pack page keeps answering something unexpected three sessions in a row,
- the open call returns an error it does not know, or a response it cannot read,
- a pack is paid with gems instead of a free booster.

It then refuses to play until you have checked your account in a browser and run `oshibot resume`.

Two cases are handled differently:

- **Expired session.** Before stopping on an unknown answer, the tool checks whether its session is still valid. If not, it asks you to run `oshibot cookie` again.
- **Network and server errors** (`5xx`). The session is simply retried later.

## 📁 Files

Everything lives next to the script and is ignored by git:

| File | Content |
|---|---|
| `session.json` | Your auth tokens. Treat it like a password. |
| `device.json` | Browser identity, presence key and device cookies |
| `profile` | Your schedule and settings |
| `config` | Discord webhook URL |
| `history.tsv` | One line per opened pack, used by `stats` and the recap |
| `rarity.tsv` | Rarity and anime of every card the tool has seen, used by the mini-games |
| `digest.tsv` | Events waiting for the next recap |
| `state`, `pause`, `halted` | Runtime state |

## ⚠️ Limitations

- **No guarantee against bans.** The schedule copies a human one, but requests still come from `curl` and from wherever you host the tool. A site that inspects TLS fingerprints or IP ranges can tell the difference.
- **Volume is a signal on its own.** Even a perfect schedule stands out if it opens 100 packs every single day. Pick the lowest preset you are happy with.
- **This repository is public.** The site's admins can read it too.
- **Free packs only.** Packs that cost gems (Prestige, Ultimate, extra banner packs) are never opened.
- **It can break at any time.** The tool relies on private endpoints observed in the browser. The server action ids are read again from the site's scripts after each update, but a deeper change may break a feature until the tool is updated.

## 🩹 The session keeps dying

You should only have to run `oshibot cookie` once. The access token lasts an hour and `oshibot` renews it by itself, for as long as the login behind it stays alive.

If the service stops after a few hours with `refresh_token_already_used`, a browser is using the **same login** as `oshibot`. Each renewal by one of them invalidates the other, and the site ends up logging both out.

The login you give to `oshibot` must belong to it alone:

1. Open a **new** private window and log in there. Do not reuse a tab that is already logged in.
2. Copy the cookie and paste it into `oshibot cookie`.
3. **Close that private window right away**, without logging out.

Your regular browser has its own login, so you can keep playing there. `oshibot cookie` warns you when the login you paste is more than 20 minutes old, which usually means it is shared with an open browser.

## 🔧 How it works

The web app opens a pack with a single `POST /api/open` call, authenticated by a Supabase session cookie. The tool rebuilds that cookie from a stored session, renews the access token through the Supabase auth API when it is about to expire, and replays the same call with a fresh `requestId` each time.

Everything else (daily login, mini-games, missions, rewards, messages, bids, listings) goes through Next.js server actions, the functions the site's buttons call. Their ids change with every site update, so `oshibot` reads them from the page's scripts and caches them until the site changes. The page data itself (missions, game state, duplicates) is read from the stream Next.js embeds in each page.

## 📄 License

[MIT](LICENSE)
