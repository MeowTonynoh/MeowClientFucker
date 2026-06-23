# Meow Client Fucker

Forensic analysis tool for Minecraft screenshare anticheat. Detects cheat clients through **RAM memory scanning** (Javaw) and **DNS cache analysis**, with generic module detection that flags suspicious behavior independently from the client.

## ⚠️ Important Notice

This tool reads process memory and DNS cache — behavior that antivirus software may flag as suspicious.

- **VirusTotal and other antiviruses may report false positives**
- Some antiviruses may block external file downloads: temporarily disable Real-Time Protection if needed

---

## How It Works

### Javaw Scanner

Scans the RAM of all active `java.exe` and `javaw.exe` processes on the system.

#### Phase 1 — Process Discovery
- Enumerates all Java processes running on the system
- Collects metadata: PID, startup time, uptime
- Opens handles with `PROCESS_VM_READ` and `PROCESS_QUERY_INFORMATION` permissions

#### Phase 2 — Memory Mapping
- Enumerates all memory regions via `VirtualQueryEx`
- Filters only committed (`MEM_COMMIT`) and readable regions
- Skips regions flagged with `PAGE_GUARD` and regions larger than 100MB

#### Phase 3 — Pattern Matching (Aho-Corasick)
- Reads memory in chunks via `ReadProcessMemory`
- Runs an **Aho-Corasick** multi-pattern search across all signatures simultaneously
- Client signatures are matched as substrings; Generic Flags use **word boundary checking** to avoid false positives
- Real-time progress bar shows scan percentage

#### Phase 4 — Classification
- **CLEAN** — nothing found
- **Client flag** — one or more client-specific strings detected → identifies the exact cheat client
- **Generic flag** — suspicious module string detected → cheat behavior found regardless of client

---

### DNS Cache Scanner

Scans the Windows DNS cache (`ipconfig /displaydns`) for domains associated with known cheat clients.

- Detects clients that phone home to their authentication or update servers
- Works even if the cheat is no longer loaded in RAM
- Correlates domains to specific clients in the database

---

## Detection: Client Flags

The tool contains signatures for **388+ cheat clients**. A client flag triggers when strings unique to a specific client are found in memory or DNS cache. Detected clients include:

**Popular / well-known**
`Vape Client` · `Meteor Client` · `LiquidBounce Client` · `Wurst` · `Salhack Client` · `Sigma Client` · `Novoware Client` · `GameSense Client` · `Osiris Client` · `Cosmos Client` · `Sorus Client` · `Azura Client` `Doomsday Client` `Novoware Client` `Argon Client` `Krypton Client` `Prestige Client` `198Macros Client`

**Crystal / Anchor PvP**
`Delta Client` · `Elysian Client` · `Onyx Client` · `Lumina Client` · `Momentum Client` · `Raven B++ Client` · `UZI Client`

**Known skids / backdoored**
`SkidBounce Client` · `Skidcraft Client` · `Backdoored Client` · `LeuxBackdoor Client` · `SalHackSkid Client` · `GrassWare.win Client` · `AllahWare Client` · `BBCWare Client`

**And many more** including: `Argon` · `Arsenic` · `Atrium` · `BleachHack` · `Caizm` · `Coffee` · `Cranberry` · `Evangelion` · `FDP` · `Fog` · `ForgeHax` · `Huzuni` · `Hydrogen` · `Ikea` · `Jex` · `Kamiblue` · `Konas` · `Kura` · `Lambda` · `LavaHack` · `Mercury` · `Mint` · `Mirai` · `NClient` · `Neptunium` · `Ozark` · `Raion` · `Rebirth` · `Rift` · `Selene` · `Seppuku` · `Silence` · `Spark` · `Swift` · `Tensor` · `Tokyo` · `Trollhack` · `Vertex` · `Vrpos` · `Xulu` · `Zeon` · `ZeroTwo` · `Zodiac` + hundreds more

---

## Detection: Generic Flags

Generic flags detect **cheat modules and behaviors** regardless of the client. These trigger when module names, descriptions, or class strings are found in memory that are typical of cheat software — even from unknown or private clients.

### Module categories detected

**Combat**
`Kill Aura` · `Crystal Aura` · `Mace Aura` · `Silent Mace` · `Aim Assist` · `Silent Aim` · `Bow Aimbot` · `TriggerBot` · `Anti-Weakness` · `Fake Punch` · `Damage Tick` · `Only Crit` · `Static HitBoxes` · `Shield Disabler` · `AntiInvis` · `W-Tap`

**Crystal / Anchor PvP**
`Auto Crystal` · `Crystal Optimizer` · `CwCrystal` · `Double Anchor` · `Anchor Exploder` · `Auto Dtap` · `MarlowAnchor` · `AntiAntiCw` · `Auto Hit Crystal`

**Totem**
`Auto Totem` · `Totem Offhand` · `Hover Totem` · `Force Totem` · `Auto Retotem` · `InventoryTotemLegit` · `Auto Double Hand`

**Movement**
`Fast Bridge` · `Bridge Assist` · `Fast Swim` · `Fast Place` · `No Break Delay` · `No Jump Delay` · `Elytra Swap` · `Elytra Glide` · `Jetpack` · `Auto Sprint` · `InventoryMove`

**Utility / Automation**
`Auto Clicker` · `Auto Pot` · `Auto Eat` · `Auto XP` · `Auto Armor` · `Auto Tool` · `Auto Mine` · `Chest Stealer` · `Shulker Dropper` · `Auto Sell` · `Cord Snapper` · `Key Pearl` · `Auto Tpa` · `Bed Macro` · `Auto Restock` · `Replace Mod`

**ESP / Vision**
`Player ESP` · `Storage ESP` · `Entity ESP` · `X-Ray` · `Health Indicators` · `Target HUD` · `Netherite Finder` · `Rtp Base Finder`

**Evasion / Anticheat bypass**
`Fake Lag` · `Ping Spoof` · `Pack Spoof` · `Stray Bypass` · `Donut SMP Bypass` · `Anti SS Tool` · `String Cleaner` · `Self Destruct` · `USN Journal Cleaner` · `Delete USN Journal` · `Generic Selfdestruct`

---

## JournalTrace Integration

After the scan, the tool offers to launch **JournalTrace** — a companion tool that analyzes the Windows USN Journal to find traces of files that were deleted or modified, useful to detect replace mods and self-destruct routines.

- Downloaded automatically from the official GitHub repository
- Launched with elevated privileges
- Filter by `.jar` to identify suspicious cheat-related files

---

## System Requirements

- Windows OS
- **Administrator privileges required** (for process memory access and USN Journal)
- .NET Runtime

Running without admin will show a warning — most features will fail without elevated permissions.

---

## Security & Privacy

- **No data is sent online** — all analysis is local
- **Memory is read-only** — the tool never modifies process memory
- **No personal data collected**
- All downloaded components (JournalTrace) come from official, verifiable GitHub repositories

---

## VirusTotal False Positives

It is normal for antivirus software to flag this tool. Technical reasons:

1. **Memory reading APIs** — use of `ReadProcessMemory` / `VirtualQueryEx` is common in both forensic tools and malware
2. **Runtime download** — downloads and executes external components
3. **Elevated privileges** — UAC prompt triggers some heuristics
4. **Obfuscated signatures** — detection strings are obfuscated to prevent bypass by cheat developers

The source code is fully private for and does not allow for independent verification.

---

## Contacts

**Creator**: Tonynoh

💬 Discord: `tonyboy90_`

📎 GitHub: [MeowTonynoh](https://github.com/MeowTonynoh)

🎥 YouTube: `tonynoh-07`
