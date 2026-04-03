# OMT-Sentinel V1.0 🛡️

**OMT-Sentinel** is an advanced, highly-secure, and fully autonomous Discord automation and utility tool designed for Pokémon catching, server activity automation, and strategic resource farming. Built natively in Python, it provides unparalleled efficiency, completely hands-free operation, and advanced account management for hardcore players.

## 🚀 Why OMT-Sentinel is Different

Most automation bots in the market are extremely basic, rely on simple image-scraping, and lack proper stealth and emergency features. **OMT-Sentinel V1.0** stands out by offering:
- **True Multi-Account Separation:** Safely decouple your "Catcher" and your "Spammer" accounts to drastically reduce detection risks.
- **Controller Infrastructure:** Control everything from a remote "Controller" account. Commands like `!pause`, `!done`, and `!stop` let you manage your fleet dynamically without touching the host PC.
- **Intelligent Incense Management:** When a Captcha drops, Sentinel pauses your fleet and dynamically issues `inc p` (pause) in your active channels. Resume seamlessly with `!done` to automatically blast `inc r` (resume).
- **Automated Logging System:** Automatically relays rare spawns, ultra beasts, paradox pokémon, and legendaries directly to your private Discord log channel automatically.
- **Self-Generating Configuration:** Launching the bot for the first time generates a heavily categorized, easy-to-read `config.json` that requires zero programming knowledge to set up.

## 🌟 Core Features

- **Blazing Fast Auto-Catcher:** Advanced asynchronous logic guarantees millisecond-precision catching.
- **Comprehensive Rare Pokémon Tracking:** Contains an exhaustive, pre-built list of Legendaries, Mythicals, Ultra Beasts, and Paradox Pokémon.
- **Smart Auto-Spammer:** Cycles through custom phrases to naturally trigger spawns in designated channels.
- **HWID Machine Lockdown:** Maximum level security integration. Sentinel uses native WMI to validate users via Hardware ID.

## ⚠️ CRITICAL SETUP: Maximum Accuracy with PokeName

To guarantee 100% catch accuracy, OMT-Sentinel relies on proper text formatting. You **MUST** invite the `PokeName` bot to your server and configure it properly.

In your server, run the following PokeName command:
```
-toggle textnaming
```
*This is required because Sentinel needs PokeName to output the Pokémon's name in pure text format for instantaneous reading and lightning-fast catching.*

## ⚙️ How to Install & Run

We have packaged the entire project into a straightforward, plug-and-play executable file for Windows!

1. Download the provided `OMT_Sentinel.exe` (Version 1.0) release.
2. Run the `.exe` file once. It will detect that it's a first-time launch and generate a `config.json` file in the same folder.
3. Open `config.json` in any text editor (like Notepad).
4. Fill in your Discord tokens, Controller ID, Log Channel ID, and Spammer settings.
5. Save the file and restart `OMT_Sentinel.exe`.

## 📌 Disclaimer
OMT-Sentinel is created for educational and proof-of-concept purposes. Users are strictly responsible for adhering to Discord's Terms of Service and respecting server rules.

---
*Built for the elite.*
