# 🛡️ OMT-Sentinel

> **Advanced Discord Automation Utility for Pokétwo — Fully Autonomous, Multi-Account, Multi-Feature.**

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=flat-square&logo=python)](https://www.python.org/)
[![discord.py-self](https://img.shields.io/badge/discord.py--self-latest-5865F2?style=flat-square&logo=discord)](https://github.com/dolfies/discord.py-self)
[![Platform](https://img.shields.io/badge/Platform-Windows-0078d4?style=flat-square&logo=windows)](https://www.microsoft.com/windows)
[![License](https://img.shields.io/badge/License-Private-red?style=flat-square)](./LICENSE)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Module Reference](#-module-reference)
  - [main.py](#mainpy)
  - [core/config.py](#coreconfigpy)
  - [core/auth.py](#coreauthpy)
  - [core/pokemon_data.py](#corepokemon_datapy)
  - [core/stats.py](#corestatspy)
  - [clients/catcher.py](#clientscatcherpy)
  - [clients/secondary.py](#clientssecondarypy)
  - [ui/dashboard.py](#uidashboardpy)
  - [utils/logger.py](#utilsloggerpy)
- [Configuration Reference](#-configuration-reference)
- [Controller Commands](#-controller-commands)
- [Pokémon Coverage](#-pokémon-coverage)
- [Setup & Installation](#-setup--installation)
- [Building from Source](#-building-from-source)
- [Data Files](#-data-files)
- [Security Architecture](#-security-architecture)
- [Disclaimer](#-disclaimer)

---

## 🔍 Overview

**OMT-Sentinel** is a Python-based, fully asynchronous Discord automation tool built for the **Pokétwo** Discord bot ecosystem. It operates two separate Discord accounts — a **Catcher** and a **Spammer/Secondary** — to maximize Pokémon catch rates while minimizing detection risk through true account decoupling, randomized delays, and intelligent captcha handling.

The tool auto-generates a `config.json` on first launch, supports HWID-locked licensing, and provides a live Rich terminal dashboard showing real-time performance metrics.

---

## 🌟 Key Features

| Feature | Description |
|---|---|
| **Dual-Account Architecture** | Completely decoupled Catcher + Spammer accounts reduce detection vector |
| **Async Auto-Catcher** | Millisecond-precision catching using `discord.py-self` + `asyncio` |
| **PokéName Integration** | Reads Pokémon names directly from `PokeName` bot output for instant catches |
| **Hint Solver (Regex Engine)** | Solves Pokétwo's hint text using a compiled `pokemon.json` database + regex |
| **Smart Captcha Handler** | Detects captcha, pauses all activity, and issues `inc p` to all active channels |
| **Remote Controller** | One designated controller account issues `!pause`, `!done`, `!stop` |
| **Incense Management** | Auto-sends `inc p` on captcha / `inc r` on resume to all tracked spawn channels |
| **Rare & Shiny Tracking** | Comprehensive hardcoded list covering Legendaries, Mythicals, UBs, and Paradox Pokémon |
| **Discord Log Channel** | Automatically relays rare catches and captcha events to a private Discord channel |
| **Live Terminal Dashboard** | `rich`-powered real-time console UI showing status, catch times, and session stats |
| **HWID Licensing** | WMI-based machine fingerprinting validated against a GitHub-hosted keystore |
| **Self-Generating Config** | First-run generates a fully annotated `config.json` — no programming knowledge needed |
| **PyInstaller Distribution** | Packaged as a single `.exe` for plug-and-play Windows deployment |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        main.py (Orchestrator)                 │
│  asyncio.gather → [CatcherClient, SecondaryClient, Dashboard] │
└──────────┬────────────────────┬─────────────────┬────────────┘
           │                    │                 │
    ┌──────▼──────┐    ┌────────▼───────┐  ┌─────▼──────┐
    │  CatcherClient│    │SecondaryClient │  │ Dashboard   │
    │ (Main Token) │    │ (Spam Token)   │  │ (rich Live) │
    └──────┬───────┘    └────────┬───────┘  └─────┬──────┘
           │                    │                 │
    ┌──────▼──────┐    ┌────────▼───────┐  ┌─────▼──────┐
    │ PokemonData │    │ Controller Cmds│  │  Stats      │
    │  (hint DB)  │    │ !done/!pause   │  │  (JSON)     │
    └─────────────┘    └────────────────┘  └────────────┘
           │
    ┌──────▼──────┐
    │   Logger    │
    │(Discord ch.)│
    └─────────────┘
```

### Async Event Flow

```
Pokétwo spawns embed  ──►  CatcherClient.on_message()
    │
    ├─ PokéName bot replies  ──►  Parse name  ──►  Send "c <name>"
    │
    ├─ Hint text detected   ──►  pokemon_db.solve_hint()  ──►  Send "c <name>"
    │
    └─ Captcha detected  ──►  handle_captcha()
           │
           ├─ Set is_paused = True
           ├─ SecondaryClient.trigger_emergency_pause()
           │      └─ Send "inc p" to all recent spawn channels
           └─ Logger.log() → Discord channel

Controller sends "!done"  ──►  SecondaryClient.on_message()
    │
    ├─ Set is_paused = False on CatcherClient
    ├─ Send "inc r" to all recent spawn channels
    └─ Reply "✅ OMT-SENTINEL RESUMED"
```

---

## 📁 Project Structure

```
omt_sentinel/
│
├── main.py                    # Entry point & async orchestrator
├── requirements.txt           # Python dependencies
├── config.json                # User config (auto-generated on first run)
├── stats.json                 # Persistent session stats (auto-generated)
├── pokemon.json               # Pokémon name database (downloaded from PokeAPI)
│
├── core/
│   ├── auth.py                # HWID validation & GitHub keystore auth
│   ├── config.py              # Config loader / schema & self-generation
│   ├── pokemon_data.py        # Pokémon DB init & hint regex solver
│   └── stats.py               # Persistent stats tracking (async I/O)
│
├── clients/
│   ├── catcher.py             # Main Discord client (catching + captcha)
│   └── secondary.py           # Spammer client + controller command handler
│
├── ui/
│   └── dashboard.py           # Rich terminal live dashboard
│
├── utils/
│   └── logger.py              # Discord log channel relay + console print
│
├── OMT_Sentinel.spec          # PyInstaller build config
├── OMT_Sentinel_V1.0.spec     # Legacy V1.0 build spec
├── OMT_Sentinel_V1.1.spec     # Legacy V1.1 build spec
└── OMT_Sentinel_V1.7.spec     # V1.7 build spec
```

---

## 📖 Module Reference

### `main.py`

The top-level async orchestrator. Responsible for:

1. **HWID Validation** — calls `validate_hwid()`, exits on failure.
2. **Pokémon DB Init** — calls `pokemon_db.initialize()` (downloads if missing).
3. **Token Validation** — checks that both tokens are non-empty and non-placeholder.
4. **Client Setup** — instantiates `CatcherClient` and `SecondaryClient`, links them.
5. **Concurrent Launch** — runs all coroutines via `asyncio.gather()`:
   - `run_catcher()` → logs in with main token
   - `run_secondary()` → logs in with spam token
   - `dashboard.start()` → starts rich live UI
   - `configure_logger()` → binds log channel once Catcher is ready

**Windows Compatibility:** Sets `WindowsSelectorEventLoopPolicy` to prevent Proactor conflicts with `discord.py-self`.

---

### `core/config.py`

**Class:** `Config`  
**Singleton:** `config`

Manages all runtime configuration by loading `config.json`. On first run, writes a heavily annotated default config and exits, prompting the user to fill it in.

#### Config Schema

```json
{
  "System": {
    "license_key": "YOUR_LICENSE_KEY_HERE"
  },
  "Accounts": {
    "catcher_account_token": "YOUR_ACCOUNT_TOKEN_HERE",
    "spammer_token":         "YOUR_SPAMMER_ACCOUNT_TOKEN_HERE",
    "controller_user_id":   "YOUR_CONTROLLER_ACCOUNT_ID_HERE"
  },
  "Channels": {
    "pokemon_spawn_channels": ["channel_id_1", "channel_id_2"],
    "log_channel_id":         "YOUR_LOG_CHANNEL_ID_HERE"
  },
  "Auto_Catcher": {
    "catch_delay_seconds_min": 1.5,
    "catch_delay_seconds_max": 3.5
  },
  "Auto_Spammer": {
    "enabled":               false,
    "spam_channel_id":       "YOUR_SPAMMER_CHANNEL_ID_HERE",
    "spam_delay_seconds_min": 5.0,
    "spam_delay_seconds_max": 10.0,
    "spam_messages": [
      "Hello there!",
      "Leveling my pokemons.",
      "What's your favorite pokemon?"
    ]
  },
  "Bot_IDs": {
    "poketwo_id":  716390085896962058,
    "pokename_id": 874910442814455838
  }
}
```

#### `Config` Attributes

| Attribute | Type | Description |
|---|---|---|
| `license_key` | `str` | User's license key for HWID auth |
| `tokens` | `dict` | `{"main": ..., "sentinel": ...}` — Discord tokens |
| `controller_user_id` | `str` | Discord user ID allowed to issue remote commands |
| `detect_channels` | `list[str]` | Channel IDs to monitor for Pokétwo spawns |
| `log_channel_id` | `str` | Channel ID for Discord event logs |
| `delay_ranges` | `list[float]` | `[min, max]` catch delay in seconds |
| `spammer_mode` | `bool` | Enable/disable auto-spammer |
| `spam_channel_id` | `str` | Channel ID where spammer sends messages |
| `spam_delay_ranges` | `list[float]` | `[min, max]` spam message delay |
| `spam_messages` | `list[str]` | Pool of messages to rotate through |
| `poketwo_bot_id` | `int` | Pokétwo bot's Discord user ID |
| `pokename_bot_id` | `int` | PokéName bot's Discord user ID |

---

### `core/auth.py`

Implements **HWID-based license validation** to prevent unauthorized use.

#### Functions

##### `get_hwid() → str`
Retrieves the machine's hardware UUID using PowerShell's `Get-CimInstance Win32_ComputerSystemProduct`.  
Returns `"UNKNOWN"` on non-Windows platforms or on failure.

##### `validate_hwid() → dict | None` *(async)*
1. Gets local HWID via `get_hwid()`.
2. Fetches the `keys.json` keystore from a private GitHub raw URL.
3. Looks up `config.license_key` in the keystore.
4. Compares the stored HWID against the local machine's HWID.
5. On **success** → returns the user's entry dict (contains `user_name`, `status`).
6. On **failure** → POSTs a registration-request webhook to Discord, then calls `sys.exit(1)`.

> **`status` field** controls account tiers. The `main()` function reads `user_entry["status"]` (e.g., `"PAID"`, `"TRIAL"`) and logs the mode on startup.

---

### `core/pokemon_data.py`

**Class:** `PokemonData`  
**Singleton:** `pokemon_db`

Manages the local Pokémon name database and the hint-solving regex engine.

#### `initialize()` *(async)*
- If `pokemon.json` is absent, calls `_download_pokemon_data()` to fetch from PokeAPI.
- Loads names into `self.names` (`list[str]`).

#### `_download_pokemon_data()` *(async, private)*
- Fetches up to 2000 Pokémon from `https://pokeapi.co/api/v2/pokemon?limit=2000`.
- Normalizes names (title-case, hyphens→spaces, special-character fixes for Pokétwo).
- Appends missing Tapu variants.
- Deduplicates, sorts, and saves to `pokemon.json`.

#### `solve_hint(hint_text: str) → list[str]`
Solves Pokétwo's hint format, e.g.:  
`"The wild pokémon is p _ _ _ _ _ u!"`

**Algorithm:**
1. Replaces `\_` with `_` (Discord markdown escape removal).
2. Extracts the pattern between `"is"` and `"!"` using regex.
3. Strips spaces and converts `_` → `.` (regex wildcard).
4. Builds `^pattern$` and compiles it.
5. Tests against every name in `self.names` (spaces stripped for matching).
6. Returns all matching names (typically just one).

#### Name Normalization Applied

| Raw (PokeAPI) | Normalized |
|---|---|
| `nidoran-m` | `Nidoran♂` |
| `nidoran-f` | `Nidoran♀` |
| `ho-oh` | `Ho-Oh` |
| `mr-mime` | `Mr. Mime` |
| `type-null` | `Type: Null` |
| `farfetch-d` | `Farfetch'd` |
| `flabebe` | `Flabébé` |

---

### `core/stats.py`

**Class:** `Stats`  
**Singleton:** `stats`

Persistent session statistics stored in `stats.json`.

#### Fields

| Field | Type | Description |
|---|---|---|
| `total_catches` | `int` | All successful catches this session |
| `rare_catches` | `int` | Catches of rare/legendary Pokémon |
| `shiny_catches` | `int` | Catches confirmed as shiny (✨ in message) |
| `total_captchas` | `int` | Number of captcha events encountered |

#### Methods

| Method | Description |
|---|---|
| `load()` | Loads stats from `stats.json` on startup |
| `save_sync()` | Synchronous write (used during `__init__` if file missing) |
| `save_async()` | Async write using `aiofiles` (called after every catch or captcha event) |

---

### `clients/catcher.py`

**Class:** `CatcherClient(discord.Client)`

The primary client. Monitors configured channels for Pokétwo activity and autonomously catches Pokémon.

#### State

| Attribute | Description |
|---|---|
| `is_paused` | Global pause flag — set to `True` on captcha |
| `recent_spawn_channels` | `set` of channel IDs where spawns have been seen (max 50) |
| `secondary_client` | Reference to `SecondaryClient` (set post-init from `main.py`) |
| `pending_catches` | `dict[channel_id → spawn_time]` — tracks unresolved spawns |
| `last_action` | Human-readable string of the last bot action (shown on dashboard) |
| `last_catch_time` | `float` seconds from spawn detection to catch confirmation |

#### Event: `on_message(message)`

Handles 6 distinct event patterns from the same message stream:

| Trigger | Condition | Action |
|---|---|---|
| **Captcha** | Author = Pokétwo; `"captcha"` or `"human"`+`"verify"` in content | `handle_captcha()` |
| **Spawn Embed** | Author = Pokétwo; embed title contains "wild" + "appear" | Track channel; store in `pending_catches` |
| **Hint Text** | `"the wild pokémon is"` in content | `solve_hint()` → catch |
| **Wrong Pokémon** | `"is the wrong pokémon"` in content | Request hint `@pokétwo h` as fallback |
| **Catch Confirm** | `"congratulations"` + `"you caught"` + bot user mentioned | Update stats, check shiny/rare, log |
| **PokéName Output** | Author = PokéName bot ID or `"poké-name"` in name | Parse text name, apply POKENAME_FIXES, catch |

#### `handle_captcha(message)` *(async)*
1. Sets `is_paused = True`.
2. Increments `stats.total_captchas` and saves.
3. Calls `logger.log()` → sends alert to Discord log channel.
4. Calls `secondary_client.trigger_emergency_pause()`.

#### RARE_POKEMON Hardcoded List

Covers the following categories:

| Category | Examples |
|---|---|
| **Gen 1-2 Legendaries** | Articuno, Zapdos, Moltres, Mewtwo, Lugia, Ho-Oh |
| **Gen 3 Legendaries** | Groudon, Kyogre, Rayquaza, Latias, Latios |
| **Gen 4-5 Legendaries** | Dialga, Palkia, Giratina, Reshiram, Zekrom, Kyurem |
| **Gen 6-8 Legendaries** | Xerneas, Yveltal, Zacian, Zamazenta, Eternatus |
| **Gen 9 Legendaries** | Koraidon, Miraidon, Ogerpon, Terapagos |
| **Mythicals** | Mew, Celebi, Jirachi, Darkrai, Genesect, Zarude, Pecharunt |
| **Ultra Beasts** | Nihilego, Buzzwole, Pheromosa, Xurkitree, Guzzlord, etc. |
| **Paradox Pokémon** | Great Tusk, Flutter Mane, Iron Valiant, Roaring Moon, etc. |

#### POKENAME_FIXES Dictionary

Corrects the hyphen-based names output from PokéName to match Pokétwo's expected format:

```python
"mr-mime" → "mr. mime"
"type-null" → "type: null"
"tapu-koko" → "tapu koko"
"great-tusk" → "great tusk"
# ... (40+ entries total)
```

---

### `clients/secondary.py`

**Class:** `SecondaryClient(discord.Client)`

The secondary (spammer) account. Handles two independent responsibilities:
1. **Auto-Spammer** — sends rotating messages to trigger Pokémon spawns.
2. **Controller Command Handler** — listens for remote commands from the configured controller.

#### Auto-Spammer

Activated on `on_ready()` if `config.spammer_mode = True`.

| Method | Description |
|---|---|
| `start_spamming()` | Creates an async task running `spam_loop()` |
| `stop_spamming()` | Cancels the task |
| `spam_loop()` | Sends random message from `spam_messages` with random delay; skips if `main_client.is_paused` |

#### Controller Commands

Handled in `on_message()` when `message.author.id == config.controller_user_id`:

| Command | Action |
|---|---|
| `!done` | Unpauses catcher; sends `inc r` to all tracked spawn channels; replies ✅ |
| `!pause` | Calls `trigger_emergency_pause()`; replies ⏸️ |
| `!stop` | Replies 🛑 and calls `sys.exit(0)` |

#### `trigger_emergency_pause()` *(async)*
1. Sets `main_client.is_paused = True`.
2. Iterates `main_client.recent_spawn_channels`.
3. Sends `@716390085896962058 inc p` to each channel via the secondary account.

---

### `ui/dashboard.py`

**Class:** `Dashboard`  
**Singleton:** `dashboard`

A `rich`-powered live terminal UI that refreshes at 2 Hz showing real-time bot status.

#### Layout

```
┌─────────────────────────────────────────────────────┐
│  🛡️ OMT-SENTINEL v1.7 | Status: RUNNING              │
├─────────────────────────────────────────────────────┤
│  Dynamic Performance Stats                           │
│  Action:           Caught a Pokémon!                 │
│  Catch Time:       2.43s                             │
│                                                      │
│  Total Catches:    142                               │
│  Rare Catches:     3                                 │
│  Shiny Catches:    1                                 │
│  Captchas:         2                                 │
└─────────────────────────────────────────────────────┘
```

**Status color:** Green when `RUNNING`, Red when `PAUSED (CAPTCHA)`.

#### `start()` *(async)*
Runs a `rich.live.Live` context, calling `generate_layout()` every 0.5 seconds.

---

### `utils/logger.py`

**Class:** `Logger`  
**Singleton:** `logger`

Dual-output logger — sends formatted text to a Discord channel and prints to the console.

#### `set_channel(channel)` 
Binds the logger to a Discord `TextChannel` object (called from `main.py` after bot is ready).

#### `log(title, message, url=None, is_rare=False, is_captcha=False)` *(async)*

Formats and sends a log entry. For captcha events, prepends `🚨 **CAPTCHA DETECTED** 🚨` with an `@everyone` ping. Always prints to local console as a fallback.

---

## ⚙️ Configuration Reference

Below is a full annotated breakdown of every `config.json` field:

```jsonc
{
  "System": {
    // Your unique license key (provided by OMT team)
    "license_key": "ABC123-XYZ456"
  },
  "Accounts": {
    // The Discord user token of the account that will CATCH Pokémon
    "catcher_account_token": "mfa.xxx...",
    // The Discord user token of the account that will SPAM to trigger spawns
    "spammer_token": "mfa.yyy...",
    // The Discord USER ID of the account you use for remote control
    "controller_user_id": "123456789012345678"
  },
  "Channels": {
    // List of channel IDs where you want Pokémon to be caught
    // Leave empty [] to catch in ALL channels the catcher can see
    "pokemon_spawn_channels": ["111111111", "222222222"],
    // A private channel ID where rare catches and captchas will be logged
    "log_channel_id": "333333333"
  },
  "Auto_Catcher": {
    // Minimum seconds to wait before sending catch command (human simulation)
    "catch_delay_seconds_min": 1.5,
    // Maximum seconds to wait before sending catch command
    "catch_delay_seconds_max": 3.5
  },
  "Auto_Spammer": {
    // Set to true to enable the secondary account spammer
    "enabled": false,
    // Channel where the spammer sends messages to trigger spawns
    "spam_channel_id": "444444444",
    // Minimum delay between spam messages (seconds)
    "spam_delay_seconds_min": 5.0,
    // Maximum delay between spam messages (seconds)
    "spam_delay_seconds_max": 10.0,
    // Rotating list of messages for the spammer to send
    "spam_messages": [
      "Looking for shinies!",
      "Anyone wanna trade?",
      "That last catch was insane lol"
    ]
  },
  "Bot_IDs": {
    // Pokétwo's official bot ID — do not change unless the bot re-releases
    "poketwo_id": 716390085896962058,
    // PokéName bot's ID — required for name-based catching (recommended)
    "pokename_id": 874910442814455838
  }
}
```

---

## 🎮 Controller Commands

Send these from your **controller account** in any channel visible to the secondary client:

| Command | Effect |
|---|---|
| `!done` | ✅ **Resume** — unpauses the catcher + sends `inc r` to all spawn channels |
| `!pause` | ⏸️ **Force pause** — sends `inc p` to all spawn channels immediately |
| `!stop` | 🛑 **Full shutdown** — terminates the entire process (`sys.exit(0)`) |

---

## 🧬 Pokémon Coverage

OMT-Sentinel covers the complete national Pokédex including:

- **All Regional Forms** (Alolan, Galarian, Hisuian, Paldean)
- **All Legendary Trios** and Duos
- **All Mythical Pokémon** (Mew through Pecharunt)
- **All Ultra Beasts** (Generation VII)
- **All Paradox Pokémon** (Generation IX, both Ancient and Future)
- **Special-character names** (Farfetch'd, Flabébé, Nidoran♂/♀, Type: Null)

The Pokémon database (`pokemon.json`) is auto-fetched from [PokeAPI](https://pokeapi.co/) on first run and supports 2000+ entries.

---

## 🚀 Setup & Installation

### Option A: Using the Pre-Built Executable (Recommended)

1. Download `OMT_Sentinel.exe` from the official release.
2. Place it in an empty folder.
3. Run it once — it will create `config.json` and exit.
4. Open `config.json` in Notepad and fill in your:
   - License key
   - Discord tokens (Catcher and Spammer)
   - Controller user ID
   - Spawn channel IDs
   - Log channel ID
5. Run `OMT_Sentinel.exe` again — you'll see HWID validation and the dashboard launch.

### Option B: Running from Source

**Prerequisites:** Python 3.10+

```powershell
# 1. Clone or navigate to the project directory
cd omt_sentinel

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run
python main.py
```

**Required: PokéName Setup**

For maximum accuracy, invite PokéName to your server and run:
```
-toggle textnaming
```
This makes PokéName output plain text names rather than formatted embeds, enabling instant parsing.

---

## 🔨 Building from Source

Requires `pyinstaller` to be installed:

```powershell
pip install pyinstaller

# Build using the latest spec file
pyinstaller OMT_Sentinel.spec
```

The output executable will be placed in `dist/OMT_Sentinel.exe`.

**Spec files by version:**

| File | Version |
|---|---|
| `OMT_Sentinel.spec` | Current / Latest |
| `OMT_Sentinel_V1.7.spec` | v1.7 |
| `OMT_Sentinel_V1.1.spec` | v1.1 |
| `OMT_Sentinel_V1.0.spec` | v1.0 |

---

## 📂 Data Files

| File | Auto-Generated | Description |
|---|---|---|
| `config.json` | ✅ Yes (first run) | All user-configurable settings |
| `stats.json` | ✅ Yes (first run) | Persistent catch/captcha counters |
| `pokemon.json` | ✅ Yes (first run) | 2000+ Pokémon names from PokeAPI |

All three files are created automatically on first launch. **Do not delete `stats.json`** unless you intend to reset your session counters.

---

## 🔐 Security Architecture

### HWID Locking

```
Local Machine
    └─ get_hwid()
          └─ PowerShell: Get-CimInstance Win32_ComputerSystemProduct .UUID
                └─ e.g. "53B191C2-648B-11ED-80F3-9C2DCDE788B0"

GitHub Raw URL (keys.json)
    └─ {
          "LICENSE_KEY": {
              "hwid": "53B191C2-...",
              "user_name": "player_name",
              "status": "PAID"
          }
       }

Validation
    └─ local_hwid == stored_hwid  ──►  ALLOW
    └─ mismatch                   ──►  POST webhook (registration req) + sys.exit(1)
```

### Token Safety

- Tokens are stored in a local `config.json` file — **never transmitted** anywhere by the tool itself.
- All Discord communication uses only the `discord.py-self` library over standard Discord Gateway WebSocket.

---

## ⚠️ Disclaimer

OMT-Sentinel is created for **educational and proof-of-concept purposes only**. Automating Discord user accounts violates [Discord's Terms of Service](https://discord.com/terms). Users are **solely responsible** for any consequences, including account termination. The developers assume no liability for misuse.

---

*Built for the elite. Use responsibly.*
