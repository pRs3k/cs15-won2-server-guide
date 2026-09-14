# Counter-Strike 1.5 Dedicated Server on WON2 — Windows 10 Setup Guide

A complete guide to running a Half-Life Dedicated Server (HLDS) with Counter-Strike 1.5 on the WON2 network, natively on Windows 10. Also works unchanged inside a Windows VM on Proxmox — see [Running This on Proxmox](#running-this-on-proxmox-as-a-windows-vm) below.

> **The critical discovery this guide documents:** HLDS2 overwrites `swds.dll` with a version that the WON2 server patcher cannot patch. Without the patch, `+sv_lan 1` makes the server LAN-only and outside players will time out when connecting. Without `+sv_lan 1`, the server tries to authenticate against dead WON servers and connections also time out. **You must restore the original 4.1.1.0 `swds.dll` and patch it with `no-won-win.exe` before anything else will work.** This is not documented in any existing guide and is the single most common reason WON2 server setups fail on Windows.

---

## Prerequisites

- Windows 10 (AMD64)
- A router with port forwarding capability
- 7-Zip installed ([https://www.7-zip.org/](https://www.7-zip.org/))
- Your PC's local LAN IP address (e.g. `192.168.0.59`)

---

## Downloads

Download all four of these before starting. Save them somewhere easy to find (e.g. your Downloads folder).

| # | File | Link |
|---|------|------|
| 1 | HLDS Standalone 4.1.1.0 | [ModDB](https://www.moddb.com/downloads/half-life-dedicated-server-4110) |
| 2 | HLDS2 (Dedicated Server 2) | [ModDB](https://www.moddb.com/downloads/half-life-dedicated-server-2) |
| 3 | Counter-Strike 1.5 Full Mod | [ModDB](https://www.moddb.com/downloads/counter-strike-15) |
| 4 | WON2 Server Patch (HLDS 1.1.1.0/4.1.1.0) | [ModDB](https://www.moddb.com/downloads/won2-patch-for-windows-hlds-11104110) |

> **Note:** The WON2 Server Patch zip contains old executables that Windows Defender will flag as malware. This is a false positive — the patcher modifies a DLL, which looks suspicious to antivirus tools. You will need to add an exclusion before extracting it (see Step 5).

---

## Step 1: Install HLDS 4.1.1.0

Run the HLDS 4.1.1.0 standalone installer. Install it to `C:\HLServer\`.

After installation, **back up the original `swds.dll`** — you will need it later:

1. Navigate to `C:\HLServer\`
2. Find `swds.dll`
3. Copy it and rename the copy to `swds.dll.original`

---

## Step 2: Install HLDS2

Extract the HLDS2 `.rar` file using 7-Zip. Inside you will find a folder called `overwritefiles` (and possibly a readme).

1. Open the `overwritefiles` folder
2. Select everything inside it
3. Copy it all into `C:\HLServer\`
4. Click **Yes / Replace** when prompted to overwrite existing files

This updates the server engine and adds WON2 protocol support, but it also overwrites `swds.dll` with a version that cannot be patched.

---

## Step 3: Restore the Original swds.dll

> **⚠️ This is the most critical step in the entire guide.** If you skip this, the WON2 patcher in Step 5 will fail with "wrong version or file already patched", and your server will either be LAN-only or unable to authenticate players. This single issue is responsible for most failed WON2 server setups on Windows.

HLDS2 overwrote `swds.dll` with a version the WON2 patcher doesn't recognize. The HLDS2 version of `swds.dll` claims to support WON2 protocol 46e, but it does not properly handle the `+sv_lan 1` bypass needed for WON2 internet play. You must restore the original 4.1.1.0 version:

1. Navigate to `C:\HLServer\`
2. Delete the current `swds.dll` (the one HLDS2 overwrote)
3. Rename `swds.dll.original` back to `swds.dll`

Alternatively, re-run the HLDS 4.1.1.0 installer into `C:\HLServer\` to restore the original file.

---

## Step 4: Install Counter-Strike 1.5

Run the CS 1.5 Full Mod installer. Install it into `C:\HLServer\`.

This populates the `cstrike` folder with all required game files: maps, models, sounds, DLLs (`dlls\mp.dll`), and configuration files including `liblist.gam` and `server.cfg`.

---

## Step 5: Apply the WON2 Server Patch

First, add an antivirus exclusion so Defender doesn't quarantine the patcher:

1. Open **Windows Security** → **Virus & threat protection**
2. Click **Manage settings** under "Virus & threat protection settings"
3. Scroll to **Exclusions** → **Add or remove exclusions**
4. Click **Add an exclusion** → **Folder** → select `C:\HLServer`

Now extract the WON2 Server Patch zip into `C:\HLServer\`. Find the file called `no-won-win.exe`.

Run `no-won-win.exe` and point it at `C:\HLServer\swds.dll`. It should report that the patch was applied successfully. If it says "wrong version or file already patched", you are still using the HLDS2-overwritten `swds.dll` — go back to Step 3.

> **What this patch does:** It modifies `swds.dll` to intercept the `+sv_lan 1` flag and redirect WON authentication to the WON2 master servers instead of the dead original WON servers. Without this patch, `+sv_lan 1` simply makes the server reject all non-LAN connections. Without `+sv_lan 1`, the server tries to authenticate players through servers that no longer exist, causing connection timeouts. **Both the patch AND the `+sv_lan 1` flag are required for WON2 internet play.**

---

## Step 6: Configure the WON2 Master Server List

The HLDS2 installation should have placed a `valvecomm.lst` file in `C:\HLServer\valve\` that already contains WON2 master server addresses. Verify its contents by opening it in Notepad — you should see entries like:

```
Master
{
    master.won2.steamlessproject.nl:27010
    master2.won2.steamlessproject.nl:27010
    ...
}
```

If the file is missing or doesn't reference WON2, download the WON2 Listing Patch from [ModDB](https://www.moddb.com/downloads/won2-listing-patch), extract `woncomm.lst`, place it in `C:\HLServer\valve\`, delete the existing `valvecomm.lst`, and rename `woncomm.lst` to `valvecomm.lst`.

---

## Step 7: Set Secure to 0

Open `C:\HLServer\cstrike\liblist.gam` in Notepad. Find the line:

```
secure "1"
```

Change it to:

```
secure "0"
```

Save the file. The original WON AntiCheat system has been offline since 2008, so this must be disabled.

---

## Step 8: Configure Your Server

Open `C:\HLServer\cstrike\server.cfg` in Notepad. Add or modify these settings:

```
hostname "Your Server Name Here"
rcon_password "pick_a_strong_password"
sv_contact "your@email.com"
mp_timelimit 30
mp_roundtime 3
mp_freezetime 3
mp_startmoney 800
mp_c4timer 35
mp_friendlyfire 0
mp_autoteambalance 1
```

Save the file.

---

## Step 9: Windows Firewall

Open PowerShell as Administrator and run:

```powershell
New-NetFirewallRule -DisplayName "HLDS WON2 UDP" -Direction Inbound -Protocol UDP -LocalPort 27015 -Action Allow
```

---

## Step 10: Router Port Forwarding

Log into your router's admin page and forward **UDP port 27015** to your server PC's local IP address (e.g. `192.168.0.59`).

If you have a double-NAT setup (e.g. an ISP gateway in front of your own router), make sure the outer gateway also forwards port 27015 to your inner router, or use DMZ/IP Passthrough on the outer gateway.

---

## Step 11: Launch the Server

Create a shortcut to `C:\HLServer\hlds.exe`. Right-click the shortcut → Properties, and set the Target to:

```
C:\HLServer\hlds.exe -console -game cstrike +map de_dust2 +maxplayers 16 -port 27015 +ip YOUR_LAN_IP +sv_lan 1 +hostname "Your Server Name Here"
```

Replace `YOUR_LAN_IP` with your PC's actual local IP address (e.g. `192.168.0.59`).

Double-click the shortcut to start the server. You should see:

```
Protocol version 46
Exe version 4.1.1.1e
...
Adding master server 195.201.142.119:27010
Adding master server 216.122.246.181:27010
```

Your server should appear on [https://won2.net/Browse-Servers/](https://won2.net/Browse-Servers/) within a couple of minutes.

---

## Optional: AMX Mod X (Admin, RockTheVote, HookMod, Rats Maps)

This adds a full plugin stack on top of the base server: admin commands, map-vote (RockTheVote), the HookMod grappling-hook plugin, custom nomination/AFK/low-gravity plugins, and a rats-only map rotation. See also [PODBot mm](#podbot-mm-bots) below for bots. Everything here is layered on top of Steps 1–11 — don't skip those first.

### Compatibility note

This server runs the original June 2002 WON build of `swds.dll`/`mp.dll` — **over a decade older** than anything the modern AMX Mod X/Metamod ecosystem is built and tested against. Two concrete consequences observed on this exact build:

- **AMX Mod X's gamedata (CRC-signature-based memory hooks) doesn't recognize this binary.** On startup you'll see `GameConfig CRC mismatch ... library "engine"` warnings, and the log will report `client_disconnected and client_remove forwards have been disabled` and `Binding/Hooking cvars have been disabled`. This is expected and non-fatal — the core plugin system, admin tools, map voting, and script-only plugins all work fine — but any plugin relying on those specific disabled forwards/hooks will be degraded.
- **Anti-cheat plugins built for ReHLDS/ReGameDLL (the modern community-rewritten CS 1.6 engine/mod binaries) will not work here**, and installing one is a real crash risk, not just a "might not work" — they read memory layouts specific to those rewritten binaries, which bear no relation to the original 2002 binary this server runs. Stick to plugins that work over the network protocol (like client-cvar queries) rather than memory scanning.

If a plugin you want doesn't load cleanly, expect to need to patch or rewrite it — the ecosystem simply wasn't built with this old a binary in mind.

### Install Metamod + AMX Mod X

1. Download and extract [Metamod v1.21.1-am](https://amxmodx.org/release/metamod-1.21.1-am.zip) — place `metamod.dll` in `C:\HLServer\cstrike\addons\metamod\dlls\`.
2. Create `C:\HLServer\cstrike\addons\metamod\plugins.ini` containing:
   ```
   win32 addons/amxmodx/dlls/amxmodx_mm.dll
   ```
3. In `C:\HLServer\cstrike\liblist.gam`, change:
   ```
   gamedll "dlls\mp.dll"
   ```
   to:
   ```
   gamedll "addons\metamod\dlls\metamod.dll"
   ```
4. Download the [AMX Mod X Base (Windows)](https://github.com/alliedmodders/amxmodx/releases/download/1.10.0.5481/amxmodx-1.10.0-git5481-base-windows.zip) and [Counter-Strike Addon (Windows)](https://github.com/alliedmodders/amxmodx/releases/download/1.10.0.5481/amxmodx-1.10.0-git5481-cstrike-windows.zip) packages. Extract both into `C:\HLServer\cstrike\` (they merge into the same `addons\amxmodx\` folder).
5. Start the server and type `meta list` then `amxx plugins` in the console to confirm both loaded.

Admin commands, admin menus, and map-vote/nextmap (`mapchooser.amxx` — this is AMX Mod X's modern merged RockTheVote/nextmap plugin) are all enabled by default in `addons/amxmodx/configs/plugins.ini` — no extra setup needed. Admins are configured in `addons/amxmodx/configs/users.ini`.

### Install HookMod

[HookMod](https://gamebanana.com/mods/39531) (grappling-hook weapon, bind a key to fire it) ships as a single self-contained AMX Mod X plugin (`adminhook.amxx`, no separate module/DLL needed):

1. Download `hook.rar`, extract, and copy `adminhook.amxx` to `cstrike\addons\amxmodx\plugins\`.
2. Add a line for it in `cstrike\addons\amxmodx\configs\plugins.ini`.

### Anti-cheat: don't bother with `query_client_cvar` on this build

An earlier version of this section documented a custom plugin (`cvarguard.sma`) that used AMX Mod X's `query_client_cvar` native to flag suspicious client cvars. **It doesn't work, and the failure is silent unless you check the logs.** `query_client_cvar` requires HLDS build 3382 or later on both server and client — this server runs build **2056** (June 2002), confirmed by grepping the exact build number out of the boot banner and cross-referencing AMX Mod X's own bug tracker. The plugin loads without error and just never fires; you won't notice unless you're watching for `Client CVAR querying is not enabled - check MM version!` in the log (which, notably, it also throws for PODBot's fake clients, which is how this got caught).

Real memory-hooking anti-cheat is off the table too, for the same reason as the ReHLDS/ReGameDLL note above — this build's gamedata doesn't support it (see the `Binding/Hooking cvars have been disabled` warning at boot).

What actually works, because it's enforced by the engine's own connection handshake rather than a plugin: native rate/update cvars in `server.cfg`. Confirmed present in this exact `swds.dll` by scanning the binary for the literal cvar strings — `sv_mincmdrate`/`sv_maxcmdrate` are **not** present (added in a later build than this one) and are omitted:

```
sv_minrate 3000
sv_maxrate 25000
sv_minupdaterate 20
sv_maxupdaterate 60
sv_cheats 0
```

This won't catch aimbots or wallhacks (nothing will, on this binary, without deep reverse-engineering work) — it's a config-abuse baseline. Combine with admin.amxx's manual kick/ban for anything a plugin can't catch.

### Custom plugins

Three small plugins, compiled locally with the `amxxpc.exe` bundled in `addons/amxmodx/scripting/` (no third-party download needed — write the `.sma`, run `amxxpc.exe yourplugin.sma`, copy the resulting `.amxx` to `addons/amxmodx/plugins/`, add it to `plugins.ini`):

- **Nominations + RockTheVote** — `nominate <mapname>` lets players nominate a map into a shared pool (max 6); `say /rockthevote` (or `say /rtv`) votes to change the map early, needs 60% of connected players, auto-fills the vote from `mapcycle.txt` if nobody nominated anything. `amx_nomvote` (admin) can also force-start a vote. 20 second vote timer, changes level to the winner. Companion to the stock `mapchooser.amxx` vote-for-nextmap system. **This is a real, working implementation** — `say /rockthevote` isn't a pre-existing AMX Mod X command, it's this custom plugin.
- **AFK Manager** — tracks player position via `get_user_origin` on a repeating timer; no movement for 60s gets a warning, 180s gets kicked. Pure polling, no forwards/hooks that this build doesn't support. **Exempts spectators/observers**: `is_user_alive()` alone doesn't distinguish "observer" from "alive playing," and an observer's camera doesn't move in world-origin terms the way a playing client's does, which caused false-positive idle kicks on anyone spectating. Fixed by checking `pev(id, pev_iuser1) != 0` (observer flag) and skipping the idle timer entirely for those players.
- **Low Gravity** — `amx_lowgravity` (admin) toggles `sv_gravity` between normal (800) and low (250). Resets to normal at the start of every map — off by default, opt-in per map. Available on request alongside HookMod (see MOTD below).

### Rats map pack

22 rats-genre maps from four sources, all in `mapcycle.txt` (the stock maps were removed entirely — this build runs rats-only):

- [Rats Map Pack](https://varq.net/en/maps/counter-strike-1.6/rats-map-pack) — `cs_rats2`, `de_rats`, `de_rats3`, `de_rats4_final`
- [Chris Spain's CS Rats Pack](https://gamebanana.com/mods/452532) (the original rats maps author's own releases) — `de_rats_2001`, `de_rats_2002`, `de_rats2_2002`, `de_rats3_2002`, `de_ratsxl`
- [de_desktop](https://gamebanana.com/mods/83585) — found by searching GameBanana's own site search directly (its general map-search API doesn't surface everything; the in-page search box does)
- A curated pull from GameBanana's full CS 1.6 map catalog (game ID `4254` — the API's own game-ID guesses are wrong, verify it against a known mod first) searched for `rats`: `de_rats_bedroom`, `de_rats_nintendo`, `de_officeratz`, `de_ratworld` (actual bsp is `de_ratworld_v1`, comes with a full custom asset folder — models/sounds/sprites/textures, not just the map), `de_outhouse`, `de_rats_italy` (actual bsp is `de_italy_rats` — filename doesn't match the mod title), `de_rats_garden_csz`, `de_rats_cave_csz`, `de_rats_aircraft_csz`, `de_the_office_rats_csz`, `cs_rats5`, `de_rats_motel`, `de_rats_apartment`

Extract each pack's `maps\*.bsp`/`*.res`/`*.txt`/`*.nav` into `cstrike\maps\` (and any `models`/`sound`/`sprites`/`gfx` folders into the matching `cstrike` subfolder, for maps like `de_ratworld_v1`/`de_desktop` that ship custom assets), and add the map names to `cstrike\mapcycle.txt` — the actual filename, not the mod's display title, for the two maps noted above where they differ.

GameBanana's file API (`https://gamebanana.com/apiv11/Mod/<id>?_csvProperties=_sName,_aFiles`) gives a direct, scriptable download URL and an AV scan result for any mod page — much more reliable than scraping the page itself, which is a heavy client-rendered SPA.

---

## PODBot mm (Bots)

[PODBot mm](https://github.com/APGRoboCop/podbot_mm) — the classic waypoint-based bot, actively maintained on GitHub, and explicitly documented as compatible with CS 1.5. This matters more than it sounds: the modern alternatives (RCBot2, YaPB) are built and tested against ReHLDS/ReGameDLL, not this vintage of binary, and carry the same crash risk discussed above for anti-cheat plugins. PODBot mm predates all of that and works as a plain Metamod plugin, no engine rewrite required.

### Install

1. Download [`podbot_full_V3B24.zip`](https://github.com/APGRoboCop/podbot_mm/releases/download/V3B24-APG/podbot_full_V3B24.zip) and extract it. The `podbot` folder inside becomes `cstrike\addons\podbot\` (the whole folder, not just the DLL — it ships with a large default waypoint set and config files you want).
2. Add a line to `cstrike\addons\metamod\plugins.ini`:
   ```
   win32 addons/podbot/podbot_mm.dll
   ```
3. Start the server and check for the POD-Bot mm version banner in the console alongside Metamod's and AMX Mod X's.

`podbot.cfg` (in the podbot folder) ships configured to auto-add 6 bots on every map start (`pb add 100` × 6) — adjust the count or remove those lines if you don't want that.

### Waypoints

PODBot needs a `.pwf` waypoint file per map to spawn bots on it — without one, it just logs `No Waypoints for this Map, can't create Bot!` and continues normally, no crash. The default download already covers most stock maps (`de_dust2`, `de_aztec`, `cs_office`, etc. — 60 waypoint files, including CS-1.5-specific variants marked `(CS 1.5)`).

For the rats maps, waypoints came from three places:

- **Bundled with the map itself** — some of the GameBanana `_csz`-suffixed rats maps ship their own `.pwf` inside the map's own download, under `addons/podbot/wptdefault/`. Worth checking every map archive's file listing for this before assuming you need to source one separately.
- **A dedicated waypoint submission** — searching GameBanana for `<mapname> waypoint` surfaces standalone waypoint uploads for specific maps (e.g. `de_rats` has one). Watch out for waypoints in the *wrong format*: some submissions labeled "waypoints" actually contain a `.nav` file — the mesh-based navigation format from the official Valve CS bot (or Source-engine games), which is a completely different system from PODBot's point-graph `.pwf` format and isn't usable here. Check the file extension before installing, not just the listing title.
- **Bulk waypoint packs** — old community collections covering thousands of maps at once occasionally include a niche map by chance. One useful source: [csgames.lt's "5600 Waypoints Pack"](https://csgames.lt/2013/10/07/downloads/5600-waypoint-pack-podbot/), hosted on MEGA (not directly scriptable — the decryption key lives in the URL fragment, which only MEGA's own client handles, so this one needs a manual download). **If you're handed an old `.exe` from a source like this, don't run it** — most of these old packs are self-extracting 7z/RAR archives with a small stub attached, and `7z.exe`'s own list/extract mode reads the archive table directly without ever executing the stub. Always list contents first (`7z l file.exe`) and check for anything other than expected data extensions (`.pwf`/`.pvi`/`.pxp` for waypoints; `.bsp`/`.res`/`.wad`/`.mdl`/`.wav`/`.spr` for maps) before extracting, let alone running.

All `.pwf` files go in `cstrike\addons\podbot\wptdefault\`. As a matter of course, this server keeps the entire unused portion of any waypoint pack it downloads (5600+ files, ~300MB — trivial disk cost) rather than just the maps currently in rotation, since a waypoint is only useful data sitting there for later, and PODBot only reads whichever one matches the current map.

### Maps installed but not in rotation

A bulk map+waypoint pack pulled from GameBanana (18 maps, none rats-themed — `de_cache`, `de_overpass`, `de_construction`, `de_vegas`, `de_impact`, `de_vengeance`, `de_virus`, `de_westwood`, `de_zima`, `de_alphacode_wi`, `de_ironblood`, `de_civic`, `de_verso`, `de_gorge`, `de_mysterious`, `de_onyx`, `de_scud`, `de_simpsons`) is installed on disk with working bot waypoints, but deliberately left out of `mapcycle.txt` to keep the rotation rats-only. Add any of them to `mapcycle.txt` whenever you want — no further setup needed, bots included.

### Bot cosmetics and difficulty

Out of the box, PODBot mm's default bot names carry a `[P0D]`/`[P*D]`/`[POD]`-style prefix and bots will occasionally send chat messages — both changed for this server:

- `cstrike\addons\podbot\botnames.txt` — every bot name rewritten with a uniform `[BOT] ` prefix (truncated where needed to fit PODBot's 21-character name limit).
- `pb_detailnames 1` → `0` and `pb_chat 1` → `0` in `podbot.cfg` — disables the extra name-decoration and bot chat.
- `pb_minbotskill`/`pb_maxbotskill` set to `1`/`30` (from the defaults of `95`/`100`) — all bots play at low difficulty.

### Spectator mode

`allow_spectators 1` is set in `server.cfg` (the correct cvar name for this build — `mp_allowspectators` doesn't exist here, that name came later). Confirmed via binary string-scan of `mp.dll` before use, per the pattern established in the anti-cheat section above: always verify a cvar exists in this exact binary before assuming it, since cvar names shifted across HLDS versions.

### Server identity and MOTD

Server is named **"Indie's Rats"** (`hostname` in `server.cfg`; also remove any `+hostname` override from the launch shortcut/scheduled task, since that flag overrides `server.cfg`). `cstrike\motd.txt` was rewritten to explain the rats theme, mention that HookMod and low gravity are both available on request, and list the RockTheVote/nomination commands above.

### Advanced Quake Sounds (AQS)

[Advanced Quake Sounds](https://github.com/ClaudiuHKS/AdvancedQuakeSounds) adds Quake-style announcer callouts (multi-kill, first blood, etc.). Installed from the project's raw GitHub files (`AQS.sma`, `AQS.ini`, `sound.zip`), compiled locally with `amxxpc.exe` like the custom plugins above — it doesn't need Orpheu or ReAPI on this build, confirmed by checking that `get_gamerules_float` (the native it actually uses) exists in `fakemeta.inc`.

Two non-fatal warnings appear at boot, consistent with the gamedata-mismatch pattern discussed in the anti-cheat section: an invalid `register_event("HLTV",...)` call, and `get_gamerules_size` disabled. The plugin still loads and runs — one minor sub-feature is degraded, not the whole thing.

### Map asset audits

Third-party map packs from GameBanana/varq.net/etc. don't always ship every custom sound/model/sprite a map's `.bsp` references — `de_rats4_final` originally shipped with three missing ambient sounds because the pack used for it was map-only. To catch this systematically across the whole map rotation (not just the one map where it was noticed), a PowerShell script scans every installed `.bsp`'s raw bytes for `.wav`/`.mdl`/`.spr` references (GoldSrc entity data stores these as plain ASCII strings) and checks whether the referenced file actually exists on disk.

Two things worth knowing if you write your own version of this:

- **Check the `valve\` fallback folder, not just `cstrike\`.** GoldSrc mods inherit shared assets from the base game's folder when the mod folder doesn't override them — a naive script that only checks `cstrike\` will report hundreds of false positives on completely untouched stock maps like `de_dust2`, because they're relying on `valve\sound\...` for shared ambience/weapon sounds.
- **Sound vs. model/sprite paths resolve differently.** A `.wav` reference in an `ambient_generic` entity is stored *relative to* `sound\`, without that prefix — you have to prepend it yourself. A `.mdl`/`.spr` reference already includes its own folder prefix (e.g. `models\props\trash.mdl`) — using it as-is. Getting this backwards produces doubled-path false positives like `models\models\hostage.mdl`.

Running the corrected script found 117 missing references across 19 of the 66 installed maps. All 117 were eventually resolved by re-downloading each map's full resource pack (not just the `.bsp`) and copying in the missing files, but a few lessons from tracking down the stubborn ones:

- **Two GameBanana mods can share a near-identical name for completely different maps.** `de_office_rats` (by one author) and `de_the_office_rats_csz` (by another) are unrelated maps that happen to share an office theme — grabbing the wrong one silently "fixes" nothing for the one you actually run, because none of the filenames match. Always search GameBanana for the map's *exact* filename, not a shortened/similar one, especially for `_csz`-suffixed maps that tend to get confused with plainer-named cousins.
- **A map can have more than one GameBanana listing, and only one might include full resources.** The `de_rats4_final.bsp` in `mapcycle.txt` is a distinct submission (`de_rats4_final`, by a different uploader) from the more commonly-linked `de_rats4` map-only pack — the map-only one is what's usually found first, but the dedicated listing had the missing ambient sounds bundled in.
- **Assets missing from every map pack you can find might belong to the base game, not the map.** `as_oilrig`/`as_tundra`/`de_alphacode_wi` were all missing the same two Apache helicopter rotor sounds (`sound/apache/ap_rotor2.wav`, `ap_rotor4.wav`) — these come from Half-Life's singleplayer `monster_apache` entity (see [apache.cpp](https://github.com/ValveSoftware/halflife/blob/master/dlls/apache.cpp)), which a dedicated-server-only install of `valve\` never ships (no `valve\sound\apache\` folder at all). Sourced from a GitHub Half-Life asset mirror instead of any map pack.
- Two lone models (`pred_plant.mdl` for `de_overpass`, `pi_bush.mdl` for `de_rats_bedroom`) and one sound (`de_impact`'s `sound/radio/elim.wav`) weren't in any map pack at all — found via a large open Counter-Strike custom-content directory index rather than GameBanana/varq.net.

---

## Auto-Restart Watchdog

A small watchdog keeps the server running unattended:

- `C:\HLServer\watchdog.ps1` checks whether `hlds.exe` is running, and if not, logs the fact and triggers a scheduled task that relaunches it.
- Registered as scheduled task `CsServerWatchdog`: runs every 5 minutes, as `SYSTEM`, logon mode `Interactive/Background` (so it runs whether or not anyone is logged into the console).
- The relaunch target, scheduled task `CsServerLaunch`, runs the actual `hlds.exe ...` launch command.

**Gotcha: `CsServerLaunch` must also be `Interactive/Background`, not `Interactive only`.** If the launch task is created as "Interactive only" (the default you get from `/it` in `schtasks /create`, useful *during setup* so you can see the console window and confirm the server boots cleanly), it silently fails to run whenever nobody is logged into the VM's console — `schtasks /run` reports `SUCCESS: Attempted to run...` even though nothing happens, and the task's `Last Run Time` never updates. This defeats the whole point of a watchdog: if the VM reboots (e.g. a Windows Update) and nothing auto-logs into the console afterward, the watchdog detects `hlds.exe` is down every 5 minutes forever but can never actually bring it back. Recreate the task without `/it` once you've confirmed the launch command works:

```powershell
schtasks /delete /tn CsServerLaunch /f
schtasks /create /tn CsServerLaunch /tr "cmd.exe /c cd /d C:\HLServer && hlds.exe -console -game cstrike +map de_rats4_final +maxplayers 16 -port 27015 +ip 192.168.0.210 +sv_lan 1" /sc onstart /ru SYSTEM /rl highest /f
```

Running as `SYSTEM` needs no stored password and works for `hlds.exe` since it doesn't need any particular user-account privileges — it's just listening on a UDP port and reading/writing its own folder.

## No HTTP Fast-Download — Pre-Sync Client Content Instead

Modern GoldSrc/Source servers can point clients at an HTTP mirror (`sv_downloadurl`) for fast, reliable resource downloads. **This build doesn't have it** — confirmed by scanning `swds.dll` for the cvar string, which isn't present at all (it was added in a later HLDS revision than this June 2002 build). The only download path available is the original slow, unreliable UDP-based resource transfer (`sv_allowdownload`/`cl_allowdownload`), which is prone to client-side timeouts, especially once you've added a lot of third-party content (custom maps bring their own models/sounds/sprites, and there's a lot of it in a rats-only rotation).

The practical fix is to avoid triggering in-game downloads at all: mirror the server's `cstrike\maps`, `cstrike\sound`, `cstrike\models`, and `cstrike\sprites` folders onto each regular player's own client install ahead of time, so their client already has everything and never needs to request it mid-connect. Re-sync after adding any new map or plugin that ships its own custom assets.

---

## Important Notes

- **`+sv_lan 1` is required.** This sounds counterintuitive, but with the patched `swds.dll`, this flag redirects the server to use WON2 master servers instead of the dead original WON auth servers. Without it, the server will try to authenticate through dead servers and outside players will time out. Without the `swds.dll` patch, this flag restricts the server to LAN only.

- **`+ip` is required.** Without specifying the IP, HLDS may bind to a virtual adapter (e.g. WSL2's 172.x.x.x interface) instead of your real LAN interface.

- **Do not use the Steam version of CS.** The WON2 network uses Protocol 46. Steam CS 1.6 uses Protocol 47/48 and is incompatible. You need the pre-Steam retail/WON version of Half-Life with CS 1.5.

- **Client setup matters too.** Players connecting to your server also need a WON2-patched Half-Life client. See the [Steamless CS Project step-by-step guide for players](https://v5.steamlessproject.nl/index.php?page=stepbystepplayer) for client setup instructions.

- **Testing from inside your own network.** Connecting to your own public IP from behind the same router may or may not work depending on whether your router supports NAT hairpinning. If it doesn't, you'll get a timeout or "LAN servers are restricted to local clients." This is a router limitation, not a server problem. Use the LAN IP to connect locally, or test from a device on a different network (e.g. phone hotspot).

- **The `custom.hpk` error is harmless.** This file stores player spray logos and is created automatically after the first player connects.

---

## Client Setup (for players joining your server)

Players need a WON2-patched Half-Life or Counter-Strike Retail client. There are two paths:

### From a Half-Life Retail CD

1. Install Half-Life from CD
2. Download and install the [Half-Life 1.1.1.0 update](http://files.steamlessproject.nl/download.php?id=13) (`hl1110.exe`)
3. Download and install the [CS 1.5 full mod package](https://www.moddb.com/downloads/counter-strike-15)
4. Download and install the [Half-Life 1.1.1.2 Retail Update](https://www.moddb.com/downloads/half-life-update-1112-patch) (includes WON2 listing patch, widescreen support, and bug fixes)

### From a Counter-Strike Retail CD

1. Install Counter-Strike Retail from CD
2. Download and install the [CS Retail update 1.0.0.5](https://contentuk.planetwon2.com/pw2/cs1005.exe) (`cs1005.exe`) — this includes CS 1.5
3. Download and install the [Half-Life 1.1.1.2 Retail Update](https://www.moddb.com/downloads/half-life-update-1112-patch)
4. Modify the installation path to be C:\Sierra\Counter-Strike\ and accept the prompt to install into the existing directory
5. Right click the icon to launch Counter-Strike > Properties > Modify "Target" to be "C:\Sierra\Counter-Strike\hl.exe" -game cstrike" 
6. Repeat this for any shortcuts to mods to have them launch using the patched hl.exe rather than the unpatched cstrike.exe
7. If using a widescreen monitor, see the following section to add widescreen support
8. Launch Counter-Strike. When propted, enter the CD key on your Counter-Strike retail case (even though it asks for a Half-Life cd key)

### Widescreen and Display Scaling

To add widescreen resolutions to the video menu, apply the [Resolution/FOV/MP3 Patch](https://www.moddb.com/downloads/half-life-won-resolution-fov-mp3-patch) on top. Use the Half-Life 1.1.1.0 version. Note: this patches `hl.exe`, so if you're launching via `cstrike.exe`, either switch to launching with `hl.exe -game cstrike` or copy the patched `hl.exe` over `cstrike.exe`.

**DPI scaling fix (important on modern high-DPI displays on modern Windows- likely not necessary in Windows XP):**

1. Right-click your game shortcut → **Properties** → **Compatibility** tab
2. Check **"Override high DPI scaling behavior"**
3. Set "Scaling performed by" to **Application**
4. Leave all other compatibility options unchecked

This prevents Windows from trying to scale the game window, which causes the display to be too large or blurry. Combined with the `-w` and `-h` launch flags, the game should render at native resolution.

### Connecting

Create a shortcut to `hl.exe` (or `cstrike.exe` if using CS Retail). Right-click → Properties and set the target to:

```
"C:\Sierra\Counter-Strike\hl.exe" -game cstrike -gl -console -32bpp -noipx
```

Replace the path with your actual install location.

**Useful client launch flags:**

| Flag | Purpose |
|------|---------|
| `-gl` | OpenGL renderer (recommended for modern systems) |
| `-d3d` | Direct3D renderer (alternative if OpenGL has issues) |
| `-w 1920 -h 1080` | Force a specific resolution (use your monitor's native resolution) |
| `-console` | Enables the developer console (tilde `~` key) |
| `-32bpp` | 32-bit color depth |
| `-noipx` | Disables IPX networking (not needed, reduces startup time) |
| `-nofbo` | Fixes some OpenGL rendering issues on modern GPUs |

A good all-purpose shortcut for widescreen:

```
"C:\Sierra\Counter-Strike\hl.exe" -game cstrike -gl -console -32bpp -noipx -w 1920 -h 1080
```

Once in-game, go to **Multiplayer → Internet Games → Update List** to browse WON2 servers, or open the console (`~`) and type:

```
connect YOUR_SERVER_PUBLIC_IP:27015
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Server doesn't appear on won2.net | Verify `valvecomm.lst` contains WON2 master server addresses. Check that UDP 27015 is forwarded on your router. |
| Players time out connecting | Ensure `swds.dll` is the patched **original 4.1.1.0** version, not the HLDS2 version. Re-apply `no-won-win.exe` if needed. |
| "LAN servers restricted to local clients" | This error means the `swds.dll` patch was not applied, so `+sv_lan 1` is restricting the server to LAN only. Restore the original 4.1.1.0 `swds.dll` and re-apply `no-won-win.exe` (see Steps 3 and 5). If this occurs when connecting from inside your own network via the public IP, it may also be a NAT hairpinning issue — try using the LAN IP instead. |
| WON2 patcher says "wrong version" | You're patching the HLDS2 version of `swds.dll`. Restore the original 4.1.1.0 `swds.dll` and try again (see Step 3). |
| HLDS binds to wrong IP (172.x.x.x) | Add `+ip YOUR_LAN_IP` to the launch arguments. |
| Defender blocks the WON2 patcher | Add `C:\HLServer` as an exclusion in Windows Security → Virus & threat protection → Exclusions. |
| Server shows on won2.net but players can't connect | Check your router port forwarding. UDP 27015 must be forwarded all the way through to your server's LAN IP. If you have double NAT (ISP gateway + your router), both layers need to pass the traffic. |
| Server listed but with wrong hostname | The `+hostname` launch parameter overrides `server.cfg`. If your `server.cfg` changes aren't taking effect, add `+hostname "Your Name"` to the shortcut target. |

> **Note:** If the Steamless Project download links are dead, [this Internet Archive collection](https://archive.org/details/hlwon_upd) has mirrors of the patches.

> **Do not use Steam.** The Steam version of Half-Life/CS uses Protocol 47/48 and is completely incompatible with WON2 (Protocol 46).

---

## Running This on Proxmox (as a Windows VM)

Everything in this guide works unchanged inside a Windows VM on Proxmox — HLDS is a native Win32 application, so virtualizing it costs nothing in compatibility. This section covers the VM-specific setup; once Windows is installed, follow Steps 1–11 above exactly as written.

### VM configuration that avoids BSODs

Windows VMs on Proxmox/KVM can be flaky with the default settings (`CRITICAL_PROCESS_DIED` crashes are a common failure mode, especially with VirtIO drivers on a fresh install). The config below is the one that installed and ran cleanly with no driver-injection step required:

```
qm create <vmid> --name cs15-won2-server \
  --memory 4096 --cores 2 --cpu host \
  --machine q35 --bios ovmf \
  --efidisk0 local-lvm:1,efitype=4m,pre-enrolled-keys=1 \
  --tpmstate0 local-lvm:1,version=v2.0 \
  --scsihw virtio-scsi-pci \
  --sata0 local-lvm:60,format=raw \
  --net0 e1000,bridge=vmbr0 \
  --ide2 local:iso/<your-windows-iso>,media=cdrom \
  --boot order=ide2
```

Key points:
- **`e1000` NIC and a `sata0` disk, not VirtIO.** VirtIO needs drivers injected during Windows Setup (via a second attached ISO) or Windows won't see a disk to install to. `e1000`/SATA are natively supported by Windows with zero extra steps, at the cost of somewhat lower theoretical throughput — irrelevant for a game server pushing UDP packets.
- **`--tpmstate0` and `--bios ovmf` (UEFI) are required for Windows 11.** Windows 10 doesn't need them, but if you only have a Windows 11 ISO handy, this works fine — HLDS has no OS-version dependency.
- **`--cpu host`**, not the default `kvm64`. The default conservative CPU type is known to cause hangs in some Windows workloads on Proxmox; `host` exposes the physical CPU's real feature set.
- **Bridge to your main LAN (`vmbr0` or equivalent)**, not an isolated/internal-only bridge. The VM needs a routable LAN IP for port forwarding to work, same as a physical PC would.
- Give the VM a **static IP** (either via Windows' network settings or DHCP reservation) so your router's port-forward rule doesn't break when the IP changes.

### The gotcha: Windows silently blocks inbound traffic on this NIC type

After first boot, Windows may bind the `e1000` adapter to the **Public** network firewall profile even if you select "Work" or "Private" during setup. This silently blocks inbound connections — including RDP, and potentially the game port rule from Step 9 if it was scoped to specific profiles — with no obvious error. `New-NetFirewallRule` succeeds, the rule shows as enabled, and it still doesn't work.

Check and fix it from an elevated PowerShell:

```powershell
Get-NetConnectionProfile   # look for NetworkCategory: Public

Set-NetConnectionProfile -InterfaceIndex <N> -NetworkCategory Private
Set-NetFirewallRule -Name "HLDS WON2 UDP" -Profile Any   # or re-create it with -Profile Any
```

If you can't get a network profile change to stick, widening the specific rule to `-Profile Any` is sufficient on its own.

### Step 10 changes slightly

Port-forward UDP 27015 to the **VM's** IP address, not the Proxmox host's IP. Everything else in Step 10 (double-NAT considerations, etc.) is unchanged.

### Why not a native Linux HLDS install instead of a Windows VM?

Half-Life has a Linux dedicated server build, and Proxmox is a Linux-first hypervisor, so running HLDS natively in an LXC container looks appealing. It doesn't work for this specific setup: the WON2 patch (Step 5) is a binary patcher (`no-won-win.exe`) that modifies the Windows `swds.dll` PE binary specifically. The Linux build uses different binaries entirely (`.so` files), and no equivalent WON2 patch exists for them. Reverse-engineering a Linux-native patch is out of scope — a Windows VM is the practical path.

### Bonus: snapshots

Once the server is fully configured (through Step 9), shut down and take a Proxmox snapshot. If a future Windows Update or misconfiguration breaks something, you can roll back in seconds instead of redoing the whole setup.

---

## Why Not Docker on WSL2?

[Ch0wW's docker-hlds-won2](https://github.com/Ch0wW/docker-hlds-won2) is a Docker image that automates the HLDS + WON2 setup using a Debian 8 i386 container. It may be a viable option on a native Linux host or VPS, but **it does not work for game servers on Windows 10 via WSL2**, and the reasons are non-obvious:

### Docker Desktop for Windows (does not work)

Docker Desktop runs containers inside its own lightweight VM, separate from WSL2's network namespace. When you use `network_mode: host`, the container shares the Docker VM's network — not the Windows host network. Heartbeat packets reach the WON2 master servers (outbound NAT works fine), but **inbound UDP from players and master server queries cannot reach the container**. The traffic path is: Internet → Windows → WSL2 VM → Docker VM → container, and inbound unsolicited UDP dies at one of these NAT boundaries.

Switching from `network_mode: host` to explicit `ports:` mapping doesn't help either — the server embeds its internal Docker bridge IP (172.18.0.x) in the heartbeat payload, which the master server can't route back to.

### Docker Engine natively in WSL2 (partially works)

Installing Docker Engine directly inside WSL2 (bypassing Docker Desktop) fixes the Docker VM layer. With `network_mode: host`, the container shares WSL2's network stack directly. Heartbeats are sent and received successfully, and `tcpdump` inside WSL2 confirms bidirectional traffic with the WON2 master servers.

However, **WSL2 on Windows 10 does not proxy inbound UDP**. Windows 10's WSL2 networking only auto-forwards inbound TCP connections, not UDP. So while the server can register with the master servers (outbound), players cannot connect (inbound UDP never reaches WSL2). There is no built-in fix for this on Windows 10. Windows 11's "mirrored" networking mode for WSL2 reportedly resolves this, but was not tested.

### Running HLDS binaries directly in WSL2 (does not work)

Extracting the 32-bit Linux HLDS binaries from the Docker image and running them directly in WSL2's Ubuntu fails because WSL2 runs a 64-bit kernel. The i386 shared libraries (`nowon.so`, `hlshield.so`, etc.) produce `wrong ELF class: ELFCLASS32` errors. While 32-bit userspace libraries can be installed via `dpkg --add-architecture i386`, the ancient glibc version these binaries require (from Debian 8 era) is incompatible with modern Ubuntu's glibc.

### Recommendation

For Windows, run HLDS natively — it's a 32-bit Windows application that runs fine on Windows 10 (and XP, 7, 8, etc.) with no compatibility layers needed.
