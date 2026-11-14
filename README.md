# Deadly Server Manager

Deadly Server Manager is a native Windows desktop application for importing, creating, running, monitoring, updating, backing up, and managing local Minecraft servers. It is built with Tauri 2, React 19, TypeScript, Rust, and SQLite, and it operates locally without a mandatory cloud account.

The application identifier is `com.deadly.servermanager`. The Deadly logo is used in the application window, sidebar, executable, installer, Start Menu shortcut, and tray.

![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11%20x64-0078D4)
![Tauri](https://img.shields.io/badge/Tauri-2-24C8DB)
![Rust](https://img.shields.io/badge/Rust-stable-B7410E)
![React](https://img.shields.io/badge/React-19-61DAFB)

## Portable workspace

By default the application uses the portable workspace `D:\Deadly Software`. The workspace contains:

```
Servers\           Imported and created server folders
Software\          Managed Paper builds downloaded from PaperMC
Backups\           Verified ZIP backups, grouped by server id
RestorePoints\     Guarded restore-point archives, grouped by server id
Downloads\         Download staging and user-visible downloads
ResourcePacks\     Imported resource-pack archives
Tools\             External tools such as the Playit agent
Data\              deadly.db, staging, file/plugin operation backups, deadlyctl token
Logs\              Application logs
Themes\            Saved appearance themes
```

Discovery order: a `deadly.portable` marker or `Data\deadly.db` beside the executable, then `D:\Deadly Software`, then `%LOCALAPPDATA%\DeadlyServerManager`. `DEADLY_WORKSPACE_ROOT` overrides discovery. When no Deadly database exists, registrations from the previous product name (`%LOCALAPPDATA%\NexusServerManager`) are adopted by copying that database into the Deadly workspace; the original folder is left untouched. The Migration Assistant also reports stale paths and can adopt or relocate registrations without touching server files.

## Features

- **Control Center** — actionable server status, readiness score, TPS/MSPT/players when actually available, plugin and update counts, backups, restore points, connection addresses, Playit state, schedules, errors/warnings, recent changes, and prominent Start / Restart / Stop controls.
- **Server Manager** — multiple registered servers, a safe server switcher, folder/registration removal that never deletes server files, and an import flow that scans read-only.
- **Server creation** — PaperMC creation wizard using the official Fill v3 API, SHA-256 verification, staging, and activation only after confirmation.
- **Server Properties** — comment- and unknown-key-preserving editor with validation, restart flags, preview, revert, compare, and an automatic restore point before saving.
- **Console & Logs** — real stdout/stderr streaming, stdin commands, level filters, search, pause and autoscroll, copy/export, timestamps, stack-trace grouping, counters, and issue detection. The Log files tab lists `latest.log`, archived `.log`/`.log.gz`, and crash reports with a date filter, per-level filtering, archive search, export, and open-location actions; historical lines without timestamps are never given synthetic times.
- **File Manager** — breadcrumb navigation, quick access to important server files and folders, search, sorting, multi-select, bulk download/delete, copy/move with paste, rename, new file/folder, drag-and-drop uploads with progress and cancel, file info, folder sizes, protected-path warnings, and automatic backups or restore points before destructive changes.
- **Players** — whitelist, operators, and ban files read from disk; live actions only when RCON is actually configured and reachable. Unavailable live data is reported as unavailable rather than fabricated.
- **Plugin Manager / Browser / Dependencies** — real JAR metadata, provider browsing (Modrinth, CurseForge with a user key, SpigotMC/Spiget with official-page fallbacks), sort by downloads, popularity, recency, rating, or name using real provider data, per-plugin source mapping with identity verification, staged and verified updates, backups before replacement, startup-report parsing, and dependency checks.
- **LuckPerms** — supported commands through RCON when available; disk data is shown as read-only when live data is unavailable.
- **PaperMC Software** — official Paper builds, Java requirement checks, verified downloads, upgrades/downgrades, stored build library, update checks, and rollback.
- **Resource Packs** — validated ZIP imports and server.properties URL/SHA-1/prompt updates. Deadly does not host files; the pack must be served over public HTTP(S).
- **Worlds** — active and archived worlds, sizes, file counts, manifests, folder actions, and guarded imports.
- **Performance / Health & Readiness** — measured CPU/memory samples and health checks with honest "unavailable" states when Windows, the process, or RCON cannot provide data.
- **Network Tester / Playit.gg** — port reachability checks and management of the official Playit agent. Account linking uses the agent's real claim URL and real login status; only Deadly-owned agent processes are started or stopped, Java and Bedrock tunnel addresses are shown separately, and downloading Playit is never faked.
- **Backup Manager** — full server, worlds only, plugins only, configurations only, **software versions**, **selected files**, and **restore points** scopes; enable/disable, schedule editor, retention, max count, location, last/next backup, and success/failure notifications. Every archive has a per-file SHA-256 manifest and is verified before being reported successful. Empty scopes fail honestly instead of producing an empty archive.
- **Restore Points** — guarded snapshots created before plugin, software, property, config, LuckPerms, world, and resource-pack changes; verification, partial-state reporting, and safety snapshot before applying a restore.
- **Configuration Manager / Server Files** — YAML/JSON/properties validation with line numbers, comparison, and a scoped premium file manager with an embedded Monaco code editor (One Dark Pro by default, tabs, unsaved indicators, search/replace, folding, auto-indent, bracket matching, word wrap, undo/redo, save as, reload, and file comparison). No file execution and no access outside the selected server or configured storage folders.
- **Maintenance Mode** — whitelist-backed enforcement with explicit reporting about what is actually active; RCON-dependent live behavior is never simulated.
- **Scheduled Tasks / Operation Manager** — exact local times and honest results; current operations, schedules, restore points, recent audit records, failures, and diagnostics.
- **Migration Assistant** — workspace inspection, stale path detection, and registration migration without touching server files.
- **External Automation** — no built-in AI model and no separate AI application. External tools drive the same backend through the local `deadlyctl.exe` bridge.
- **Activity History** — real audit records from the GUI, scheduled tasks, restore points, and CLI operations. Credentials and API keys are never recorded.
- **Appearance & Settings** — theme presets, colors, density, typography, wallpaper upload (PNG/JPG/WEBP, copied into the app-managed theme folder with opacity, blur, overlay, and position), particles with density/speed/disable controls, consistent non-distracting hover states, sidebar pin/unpin with arrow-based reorder, collapsible groups, remembered layout, tray/background behavior, autostart, Java and memory defaults, startup validation, and credential management.

## deadlyctl — local management CLI

`deadlyctl.exe` ships beside the application and talks to the running app over the local Windows named pipe `\\.\pipe\deadly-server-manager`. Requests authenticate with a random token stored in `<workspace>\Data\deadlyctl.token`. The token is created by the app with restrictive permissions and is never exposed to the web frontend. Every command uses the same backend and confirmations as the GUI and is recorded in Activity History.

```text
deadlyctl server list
deadlyctl server status <id>
deadlyctl server start|stop|restart|force-stop <id>
deadlyctl console send <id> <command>
deadlyctl properties show <id>
deadlyctl properties set <id> motd=Hello max-players=20
deadlyctl plugins list <id>
deadlyctl plugins check-updates <id>
deadlyctl plugins report <id>
deadlyctl luckperms groups <id>
deadlyctl backup create <id> [--scope full|world|plugins|config|software|selected|restore-points] [--note text] [--paths path1,path2]
deadlyctl backup list <id>
deadlyctl restorepoint create <id> [--categories config,plugins] [--reason text]
deadlyctl restorepoint list <id>
deadlyctl config validate <id>
deadlyctl config diff <id> --path server.properties [--proposed-file file]
deadlyctl health <id>
deadlyctl performance summary <id> [--limit 30]
deadlyctl activity list [--server <id>] [--limit 50]
deadlyctl diagnostics export <id> --format json|zip|html --destination <file> [--include-network] --confirmed
deadlyctl worlds list <id>
deadlyctl players list <id>
deadlyctl playit status|start|stop
```

### Full backend access for external AI

`deadlyctl call <command>` exposes the complete management surface used by the GUI, so an external AI can work without a separate API server. Every argument is a `key=value` pair and values are automatically typed (`true`, `false`, numbers, JSON arrays/objects). Use `--args-file` for large JSON payloads and `--content-file` for file contents.

```text
deadlyctl call files.list id=1 path=plugins --json
deadlyctl call files.read id=1 path=server.properties --json
deadlyctl call files.write id=1 path=server.properties confirmed=true --content-file new-server.properties
deadlyctl call files.rename id=1 path=old.yml newName=new.yml confirmed=true
deadlyctl call files.delete id=1 path=plugins/OldPlugin.jar confirmed=true
deadlyctl call files.search id=1 query=Essentials searchContents=true
deadlyctl call logs.list id=1
deadlyctl call logs.read id=1 name=latest.log lines=500
deadlyctl call logs.search id=1 name=2026-01-01-1.log.gz query=ERROR
deadlyctl call plugins.install id=1 provider=modrinth projectId=<project> confirmed=true
deadlyctl call plugins.stage-update id=1 pluginId=12
deadlyctl call plugins.apply-update id=1 pluginId=12 confirmed=true
deadlyctl call plugins.uninstall id=1 pluginId=12 confirmed=true
deadlyctl call plugins.map id=1 pluginId=12 provider=modrinth projectId=<project>
deadlyctl call players.action id=1 action=whitelist_add player=Steve
deadlyctl call players.action id=1 action=kick player=Steve reason="Restarting"
deadlyctl call luckperms.action id=1 action=create_group group=vip
deadlyctl call luckperms.action id=1 action=group_permission_set group=vip node=essentials.fly value=true
deadlyctl call worlds.activate id=1 name=world_nether
deadlyctl call config.apply id=1 path=config.yml proposed="<yaml>" confirmed=true
deadlyctl call tasks.create id=1 --args-file task.json
deadlyctl call maintenance.enable id=1 message="Updating plugins" kickPlayers=true confirmed=true
deadlyctl call network.diagnose id=1 --json
deadlyctl call backup.restore id=1 backupId=3 confirmed=true
deadlyctl call restorepoint.apply id=1 restorePointId=8 confirmed=true
```

`deadlyctl call --help` and `deadlyctl` (without arguments) print the full command list. Destructive operations require `confirmed=true`; backend guards reject uninspected or unsafe operations the same way the GUI does. Use `--json` for machine-readable output and check the process exit code.

Examples:

```text
deadlyctl --workspace "D:\Deadly Software" health 1
deadlyctl backup create 1 --scope selected --paths plugins/EssentialsX,server.properties --note "Before plugin update"
deadlyctl diagnostics export 1 --format zip --destination "D:\Deadly Software\Downloads\server-1-diagnostics.zip" --confirmed
```

The CLI reports `UNAUTHORIZED` for a mismatched token and a clear connection error when the desktop app is not running. It cannot access arbitrary filesystem paths; backup `--paths` are validated inside the server folder.

## Requirements

- Windows 10 or Windows 11 (x64)
- Microsoft Edge WebView2 Runtime (included with current Windows 10/11)
- Java for the servers you run (Paper 1.21.x requires Java 21; newer versions may require Java 25)
- For development: Node.js 20+, Rust stable with the `x86_64-pc-windows-gnu` target, and a MinGW-w64 toolchain. This project is configured for WinLibs POSIX/MSVCRT paths in `.cargo/config.toml`; adjust those paths if your toolchain is installed elsewhere.

## Development

```bash
npm install
npm run tauri dev
```

Unit tests:

```bash
npm test
cd src-tauri && cargo test --lib
```

## Production build

```bash
npm run tauri build
```

The NSIS installer is written under the configured Cargo target directory:

```
D:\NexusBuildTarget\x86_64-pc-windows-gnu\release\bundle\nsis\Deadly Server Manager_0.1.0_x64-setup.exe
```

The installed application runs standalone and does not require Vite, Node.js, or Cargo.

## Data and credentials

- Application database: `<workspace>\Data\deadly.db` (SQLite, WAL, automatic corruption recovery with the old file preserved).
- Backups: `<workspace>\Backups\<server-id>\`.
- Restore points: `<workspace>\RestorePoints\<server-id>\`.
- Credentials (RCON passwords, CurseForge API key) are encrypted with Windows DPAPI before storage. API keys are never returned to the frontend.
- Server folders are never relocated by Deadly.

## Safety behavior

- The Minecraft EULA is never accepted automatically. Starting a server whose `eula.txt` is not accepted returns an explicit error and a review link.
- Existing start scripts (`.bat`, `.sh`) are detected and shown for review but are never executed.
- Only Java processes launched by Deadly can be stopped or force-stopped; externally started servers are inspect-only unless RCON is configured.
- Graceful stop is attempted before force stop becomes available; force stop requires explicit acknowledgment.
- Downloads go to staging, are verified against the provider SHA-256, and are atomically moved into place. An active server JAR is never replaced by an incomplete download.
- Plugin JARs are never replaced while the server is running.
- Backups are verified against a manifest before being reported successful; restores create a safety backup first.
- Empty backup scopes fail with a clear error instead of creating an unverifiable archive.
- Diagnostics are previewed and redacted before export; tokens and passwords are not included.

## Testing

See [docs/TESTING.md](docs/TESTING.md) for the verification report, including live PaperMC/Modrinth tests, process-control scenarios, backup scopes, CLI behavior, and known unverified items. See [docs/IMPLEMENTED.md](docs/IMPLEMENTED.md) for the requirement-by-requirement status.

## Project layout

```
src/                     React UI (pages, components, API client, styles)
src-tauri/src/           Rust backend
  commands/              Tauri command surface
  process/               Process manager, ownership detection, RCON, log parsing
  plugins/               Modrinth, CurseForge, Spigot providers and JAR metadata
  minecraft/             Server/plugin JAR inspection, Java probing, scanning
  backups.rs             Backup creation, verification, restore, scopes
  restore.rs             Restore points and backup import
  ipc.rs                 Named-pipe bridge used by deadlyctl
  bin/deadlyctl.rs       Local management CLI
  database.rs            SQLite schema, migrations, persistence
  scheduler.rs           Automatic backup and update-check scheduler
  secrets.rs             Windows DPAPI credential storage
src-tauri/tauri.conf.json Bundle and window configuration
src-tauri/binaries/      deadlyctl sidecar bundled with the app
docs/                    Testing and implementation reports
```

## Version

0.1.0
