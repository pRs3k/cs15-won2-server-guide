# Counter-Strike 1.5 Dedicated Server on WON2 — Windows 10 Setup Guide

A complete guide to running a Half-Life Dedicated Server (HLDS) with Counter-Strike 1.5 on the WON2 network, natively on Windows 10.

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

Now extract the WON2 Server Patch zip into `C:\HLServer\`. The zip contains two executables:

- `no-won-win.exe` — **Use this one.** This is the patcher for the original 4.1.1.0 `swds.dll`.
- `no-won-win4111.exe` — Do NOT use this. This is for a different version and will report "wrong version" on the 4.1.1.0 file.

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

## Important Notes

- **`+sv_lan 1` is required.** This sounds counterintuitive, but with the patched `swds.dll`, this flag redirects the server to use WON2 master servers instead of the dead original WON auth servers. Without it, the server will try to authenticate through dead servers and outside players will time out. Without the `swds.dll` patch, this flag restricts the server to LAN only.

- **`+ip` is required.** Without specifying the IP, HLDS may bind to a virtual adapter (e.g. WSL2's 172.x.x.x interface) instead of your real LAN interface.

- **Do not use the Steam version of CS.** The WON2 network uses Protocol 46. Steam CS 1.6 uses Protocol 47/48 and is incompatible. You need the pre-Steam retail/WON version of Half-Life with CS 1.5.

- **Client setup matters too.** Players connecting to your server also need a WON2-patched Half-Life client. See the [Steamless CS Project step-by-step guide for players](https://v5.steamlessproject.nl/index.php?page=stepbystepplayer) for client setup instructions.

- **You cannot test external connectivity from inside your own network.** Connecting to your own public IP from behind the same router will fail with "LAN servers are restricted to local clients." This is a NAT hairpinning limitation, not a server problem. Use the LAN IP to connect locally, or test from a device on a different network (e.g. phone hotspot).

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
2. Download and install the [CS Retail update 1.0.0.5](http://files.steamlessproject.nl/download.php?id=11) (`cs1005.exe`) — this includes CS 1.5
3. Download and install the [Half-Life 1.1.1.2 Retail Update](https://www.moddb.com/downloads/half-life-update-1112-patch)

### Connecting

Launch the game, go to **Multiplayer → Internet Games → Update List** to browse WON2 servers, or open the console (`~`) and type:

```
connect YOUR_SERVER_PUBLIC_IP:27015
```

### Widescreen Support

If the 1.1.1.2 patch doesn't add widescreen resolutions to the video menu, apply the [Resolution/FOV/MP3 Patch](https://www.moddb.com/downloads/half-life-won-resolution-fov-mp3-patch) on top. Use the 1.1.1.0 version. Note: this patches `hl.exe`, so if you're launching via `cstrike.exe`, either switch to launching with `hl.exe -game cstrike` or copy the patched `hl.exe` over `cstrike.exe`. You may also need to right-click the shortcut → Properties → Compatibility → check "Override high DPI scaling behavior" → set to "Application".

> **Note:** If the Steamless Project download links are dead, [this Internet Archive collection](https://archive.org/details/hlwon_upd) has mirrors of the patches.

> **Do not use Steam.** The Steam version of Half-Life/CS uses Protocol 47/48 and is completely incompatible with WON2 (Protocol 46).

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Server doesn't appear on won2.net | Verify `valvecomm.lst` contains WON2 master server addresses. Check that UDP 27015 is forwarded on your router. |
| Players time out connecting | Ensure `swds.dll` is the patched **original 4.1.1.0** version, not the HLDS2 version. Re-apply `no-won-win.exe` if needed. |
| "LAN servers restricted to local clients" | You're connecting from inside your own network via the public IP. Use the LAN IP instead, or test from an external network. |
| WON2 patcher says "wrong version" | You're patching the HLDS2 version of `swds.dll`. Restore the original 4.1.1.0 `swds.dll` and try again. Use `no-won-win.exe`, not `no-won-win4111.exe`. |
| HLDS binds to wrong IP (172.x.x.x) | Add `+ip YOUR_LAN_IP` to the launch arguments. |
| Defender blocks the WON2 patcher | Add `C:\HLServer` as an exclusion in Windows Security → Virus & threat protection → Exclusions. |
| Server shows on won2.net but players can't connect | Check your router port forwarding. UDP 27015 must be forwarded all the way through to your server's LAN IP. If you have double NAT (ISP gateway + your router), both layers need to pass the traffic. |
| Server listed but with wrong hostname | The `+hostname` launch parameter overrides `server.cfg`. If your `server.cfg` changes aren't taking effect, add `+hostname "Your Name"` to the shortcut target. |

---

## Why Not Docker on WSL2?

If you're a Linux user, [Ch0wW's docker-hlds-won2](https://github.com/Ch0wW/docker-hlds-won2) project is an excellent turnkey Docker image that handles the entire HLDS + WON2 setup automatically using a Debian 8 i386 container. On a native Linux machine, it works perfectly.

However, **running this Docker image on Windows 10 via WSL2 does not work for game servers**, and the reasons are non-obvious:

### Docker Desktop for Windows (does not work)

Docker Desktop runs containers inside its own lightweight VM, separate from WSL2's network namespace. When you use `network_mode: host`, the container shares the Docker VM's network — not the Windows host network. Heartbeat packets reach the WON2 master servers (outbound NAT works fine), but **inbound UDP from players and master server queries cannot reach the container**. The traffic path is: Internet → Windows → WSL2 VM → Docker VM → container, and inbound unsolicited UDP dies at one of these NAT boundaries.

Switching from `network_mode: host` to explicit `ports:` mapping doesn't help either — the server embeds its internal Docker bridge IP (172.18.0.x) in the heartbeat payload, which the master server can't route back to.

### Docker Engine natively in WSL2 (partially works)

Installing Docker Engine directly inside WSL2 (bypassing Docker Desktop) fixes the Docker VM layer. With `network_mode: host`, the container shares WSL2's network stack directly. Heartbeats are sent and received successfully, and `tcpdump` inside WSL2 confirms bidirectional traffic with the WON2 master servers.

However, **WSL2 on Windows 10 does not proxy inbound UDP**. Windows 10's WSL2 networking only auto-forwards inbound TCP connections, not UDP. So while the server can register with the master servers (outbound), players cannot connect (inbound UDP never reaches WSL2). There is no built-in fix for this on Windows 10. Windows 11's "mirrored" networking mode for WSL2 reportedly resolves this, but was not tested.

### Running HLDS binaries directly in WSL2 (does not work)

Extracting the 32-bit Linux HLDS binaries from the Docker image and running them directly in WSL2's Ubuntu fails because WSL2 runs a 64-bit kernel. The i386 shared libraries (`nowon.so`, `hlshield.so`, etc.) produce `wrong ELF class: ELFCLASS32` errors. While 32-bit userspace libraries can be installed via `dpkg --add-architecture i386`, the ancient glibc version these binaries require (from Debian 8 era) is incompatible with modern Ubuntu's glibc.

### Recommendation

For Windows, run HLDS natively — it's a 32-bit Windows application that runs fine on Windows 10 (and XP, 7, 8, etc.) with no compatibility layers needed. For Linux, use Ch0wW's Docker image on a native Linux host or a VPS with a real public IP.
