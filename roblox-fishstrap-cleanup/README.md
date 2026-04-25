# Removing Fishstrap Custom Skies After Uninstall (Roblox on Windows)

## The Problem

You uninstalled **Fishstrap** (a third-party Roblox bootstrapper, fork of Bloxstrap), but custom skybox textures it provided are still showing up in Roblox games such as **Roblox Rivals**. The default Roblox skies are missing — instead, you see the custom skies that Fishstrap shipped with.

This happens because uninstalling Fishstrap from "Add or Remove Programs" leaves three things behind:

1. **Fishstrap config folder** under `%LOCALAPPDATA%\Fishstrap\` — the launcher's settings.
2. **Fishstrap log folder** under `%LOCALAPPDATA%\Temp\Fishstrap\` — launch logs.
3. **Roblox's asset cache** under `%LOCALAPPDATA%\Roblox\rbx-storage\` — this is the real culprit. It holds copies of every asset your Roblox client has ever downloaded, indexed by content hash. If Fishstrap injected custom skybox assets into this cache, they survive the uninstall and keep being served to any game that requests those skybox asset IDs.

> Note: the Fishstrap mod system does **not** modify the sky textures inside Roblox's `Versions\version-*\content\sky\` folders. Those are usually still the originals — verifying their hashes confirmed this. The custom skies live in the asset cache.

## Who This Is For

Roblox players on **Windows 10/11** who:
- Installed Fishstrap (or any similar Roblox bootstrapper that supports texture mods, such as Bloxstrap)
- Uninstalled it
- Still see leftover custom textures (skies, etc.) in games

## Prerequisites

Before starting:

1. **Uninstall Fishstrap** via Settings → Apps → Installed apps → Fishstrap → Uninstall. Make sure it's actually gone.
2. **Close Roblox completely.** Important: Roblox spawns background helper processes (`RobloxPlayerBeta.exe`, `RobloxCrashHandler.exe`) that often survive after you close the game window. They will lock the cache files and prevent deletion.

## Step 1 — Verify No Roblox/Fishstrap Processes Are Running

Open **PowerShell** (Windows key, type "powershell", Enter) and run:

```powershell
Get-Process | Where-Object { $_.ProcessName -match 'roblox|rbx|fishstrap' } | Select-Object ProcessName, Id
```

If anything is listed, kill it. Either close it from Task Manager, or run:

```powershell
Get-Process | Where-Object { $_.ProcessName -match 'roblox|rbx|fishstrap' } | Stop-Process -Force
```

Re-run the first command to confirm nothing is left.

## Step 2 — (Optional) Confirm Sky Files in the Roblox Versions Folder Are Defaults

This is a sanity check. Roblox keeps its installed game files under `%LOCALAPPDATA%\Roblox\Versions\version-*\`. The sky textures live in `content\sky\`. If you have two version folders (one from Fishstrap's managed install, one from a fresh Roblox download), you can hash and compare:

```powershell
$versions = Get-ChildItem "$env:LOCALAPPDATA\Roblox\Versions" -Directory | Where-Object Name -like "version-*"
foreach ($v in $versions) {
    $sky = Join-Path $v.FullName "content\sky\sun.jpg"
    if (Test-Path $sky) { Get-FileHash $sky -Algorithm MD5 | Format-Table Hash, Path -AutoSize }
}
```

If the hashes match across versions, the sky files on disk are stock Roblox defaults and you don't need to touch them. If they differ, the file in the older (Fishstrap-managed) version may have been replaced — but in practice the asset cache is the more common culprit, so try Step 3 first.

## Step 3 — Delete Fishstrap Residual Folders

```powershell
Remove-Item -Path "$env:LOCALAPPDATA\Fishstrap" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:LOCALAPPDATA\Temp\Fishstrap" -Recurse -Force -ErrorAction SilentlyContinue
```

These hold only config and log files — safe to delete unconditionally.

## Step 4 — Clear the Roblox Asset Cache

This is the step that actually fixes the custom skies. Roblox will rebuild the cache from scratch on next launch (first game load will be slower while it redownloads assets):

```powershell
Remove-Item -Path "$env:LOCALAPPDATA\Roblox\rbx-storage" -Recurse -Force
Remove-Item -Path "$env:LOCALAPPDATA\Roblox\rbx-storage.db" -Force
Remove-Item -Path "$env:LOCALAPPDATA\Roblox\rbx-storage.db-shm" -Force
Remove-Item -Path "$env:LOCALAPPDATA\Roblox\rbx-storage.db-wal" -Force
```

If any of these fail with **"the process cannot access the file because it is being used by another process"**, a Roblox background process is still alive. Go back to Step 1, kill it, then retry.

You may notice these two entries remain after deletion — leave them, they are unrelated and tiny:
- `rbx-storage.id` — an 8-byte client identifier
- `rbx-storage-sc\` — an empty internal folder

## Step 5 — Verify Cleanup

```powershell
Test-Path "$env:LOCALAPPDATA\Fishstrap"                  # should print: False
Test-Path "$env:LOCALAPPDATA\Temp\Fishstrap"             # should print: False
Test-Path "$env:LOCALAPPDATA\Roblox\rbx-storage.db"      # should print: False
(Get-ChildItem "$env:LOCALAPPDATA\Roblox\rbx-storage" -ErrorAction SilentlyContinue).Count  # should print: 0 (or path missing)
```

## Step 6 — Launch Roblox and Test

1. Launch Roblox normally (from Microsoft Store / desktop shortcut / website "Play" button — **not** Fishstrap).
2. Join the affected game (e.g. Roblox Rivals).
3. The first load will take noticeably longer than usual — this is the cache rebuilding. Normal.
4. Look at the sky.

### Expected outcomes

- **Sky now looks default / different** — Fishstrap was injecting custom skybox assets into the cache, and clearing it worked. Done.
- **Sky still looks "custom"** — it isn't really custom; it's the game's own skybox set by the developers via `Lighting.Sky`. There is nothing on your machine to remove. Compare against another player's screenshot to confirm. Many Roblox games ship with their own custom skies and that is just how they are designed to look.

## Troubleshooting

**"The process cannot access the file"** when running Step 4
- A Roblox background process is still holding the SQLite database open. Run Step 1 again and force-kill any `Roblox*` or `rbx*` process. Common stragglers: `RobloxPlayerBeta`, `RobloxCrashHandler`. WebView2 (`msedgewebview2`) processes are normal Windows components and unrelated — leave them alone.

**Custom skies are back after one game session**
- Make sure you launched Roblox via the official launcher, not via Fishstrap (which you should have uninstalled in the prerequisites). If Fishstrap's executable is still anywhere on disk and you launched it, it will recreate everything. Check for shortcuts on your desktop, taskbar, and Start Menu.

**Nuclear option — full Roblox reset**
If something is still off, uninstall Roblox via Settings → Apps, then delete the entire `%LOCALAPPDATA%\Roblox\` folder, then reinstall Roblox from the Microsoft Store or roblox.com. This guarantees zero residuals at the cost of redownloading the client (~500 MB) and any saved local settings.

## What This Does Not Touch

- **Roblox account data** (friends, inventory, robux, game progress) — all stored server-side, untouched.
- **Other game caches or saves** — Roblox doesn't store gameplay saves locally; everything is on Roblox's servers.
- **Other Windows apps** — only Roblox-specific cache files in your user profile are removed.

The only side effect is that Roblox will redownload game assets the next time you join each game.

## Progress Log

- Updated: April 25, 2026
- Worked together to identify that Fishstrap residuals and Roblox asset cache were causing custom sky textures to persist after uninstall.
- Confirmed the fix path: remove `%LOCALAPPDATA%\Fishstrap`, `%LOCALAPPDATA%\Temp\Fishstrap`, and Roblox cache files under `%LOCALAPPDATA%\Roblox\rbx-storage`.
- Added this README update so the recovery steps and our troubleshooting progress are documented for future reference.
- Next step: verify GitHub CLI authentication and use `gh` for future repo operations.
