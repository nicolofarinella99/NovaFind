<h1 align="center">NovaFind</h1>

<p align="center">
  <b>Blazing-fast, Spotlight-style file search for Windows.</b><br>
  Local drives and SMB network shares · instant results · one portable .exe
</p>

<p align="center">
  <a href="README.it.md">🇮🇹 Leggi in italiano</a>
</p>

---

Windows search is slow, and on company network shares it often doesn't work at all. **NovaFind** stays in the background, opens with a keyboard shortcut in a few milliseconds and finds files and folders while you type, on your PC and on the servers you add.

- ⚡ **Instant**: in-memory index, results in 1–3 ms across hundreds of thousands of files
- 🗂️ **Local and network**: drives, single folders, `\\server\share` or `\\192.168.x.x\share`, with optional credentials
- 🪟 **Spotlight-style**: compact window, keyboard-first, real Windows icons, previews
- 🎨 **Glass UI with three themes**: Space (stars, aurora, shooting stars), Dark and Light, plus accent colors and adjustable transparency
- 📦 **One portable file**: ~400 KB, no installer, no DLLs, no runtime to download
- 🔒 **Private**: never connects to the internet; no telemetry, no update checks

> The interface is currently in **Italian**.

## Download

Grab `NovaFind.exe` from the [Releases](../../releases) page and double-click it.

**Requirements:** Windows 10 or 11 (64-bit), with .NET Framework 4.8 (already included in Windows 10 1903+ and Windows 11).

> The executable is not code-signed, so on first launch Windows SmartScreen may show *"Windows protected your PC"*. Click **More info → Run anyway**.

## Usage

1. Launch `NovaFind.exe`. It stays in the system tray (right-click for *Apri / Impostazioni / Esci*).
2. Press **Ctrl+Alt+Space** (configurable). If another app already uses it, NovaFind falls back to Alt+Shift+Space or Ctrl+Alt+F and tells you.
3. Type. Results update on every keystroke.

### Keyboard

| Key | Action |
|---|---|
| ↑ ↓ PgUp PgDn | move through results |
| Enter | open |
| Ctrl+Enter | show in folder (file selected in Explorer) |
| Alt+Enter | properties |
| Ctrl+C | copy path |
| Tab | add/remove the folder currently open in File Explorer |
| Ctrl+1 / 2 / 3 | all / files only / folders only |
| Ctrl+P | toggle preview pane |
| Esc | close |

Right-click a result for *Open, Show in folder, Copy path, Copy name, Properties*. Results can also be dragged into other apps.

### Search syntax

| Query | Meaning |
|---|---|
| `invoice 2024` | all words must appear in the name, in any order |
| `ext:pdf` · `ext:pdf,docx` · `*.pdf` | filter by extension |
| `dir:` / `file:` | folders only / files only |
| `"exact phrase"` | match words including spaces |

Ranking puts exact names first, then names starting with the query, then recent and frequently opened files.

### Where to search

The chips under the search bar choose where to look: **Whole PC, Desktop, Documents, Downloads**, your own folders and network servers. You can select several at once, and Ctrl+click searches only that one. When a File Explorer window is in front, a **Current folder** chip appears; it can be combined with the others or used alone.

**+ Add** opens a dialog for name, path, optional user/domain/password, whether to build an index, how often to re-index and how many parallel reads to allow on the server, plus a *Test connection* button.

## Appearance

Use the **palette button** in the search window or *Settings → Aspetto*. Changes apply live.

- **Theme:** Automatic (follows Windows light/dark), Space, Dark, Light
- **Accent color:** theme default or one of 10 colors, automatically adjusted to stay readable
- **Background transparency:** 0–100% slider, with optional frosted-glass blur
- **Animations:** full, light (transitions only) or none. They pause when the window is hidden.
- **Window size:** compact, normal or wide, with optional preview pane

## How it works

NovaFind is written in **C# with WPF on .NET Framework 4.8**, compiled with the `csc.exe` that ships with Windows. No NuGet packages, no third-party libraries.

- **Compact index:** no object per file, only parallel arrays and a single byte buffer for names (Latin-1 when possible). That's about 40 bytes per entry, and it is cached on disk so it loads in about 10 ms at startup.
- **Parallel scanning** with `FindFirstFileEx` (large fetch), at low I/O priority for background rescans. Local folders stay up to date through `FileSystemWatcher`. Optional NTFS MFT reading when run as administrator.
- **Parallel search** over the index with unsafe, allocation-free matching. Locations without an index use **live streaming search**, with results appearing as they are found.
- **Gentle with servers:** limited concurrency, reachability checks with timeouts, clear error messages, and re-indexing only on request or at the interval you choose.
- **Native Windows integration** through P/Invoke: shell icons and thumbnails, `SHOpenFolderAndSelectItems`, `WNetAddConnection2`, global hotkey, composition blur.

### Performance

Measured on a test machine (Windows 11, 8 threads, SSD, standard user):

| | |
|---|---|
| Window open (hotkey → first frame) | 10–30 ms |
| Indexing ~330,000 entries | ~1.4 s (warm file system cache) |
| Loading the index at startup | ~10 ms |
| Query on the index | 1–3 ms typical, ~13 ms for a single letter |
| Idle RAM (working set) / CPU | ~25 MB / 0% |

## Building from source

```bash
build.bat
```

That's it: it runs offline using only the C# compiler included in Windows (`%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\csc.exe`). The script first draws the logo in code and writes the `.ico` (`tools\IconGen.cs`), then compiles `dist\NovaFind.exe`.

All visual parameters (palettes, accent colors, fonts, radii, sizes, animation timings, starfield) live in [`src/Theme.cs`](src/Theme.cs).

```
src/
  App.cs              startup, single instance, tray, hotkey, benchmark mode
  Theme.cs            themes and all visual parameters
  Logo.cs             vector logo and .ico writer
  FileIndex.cs        compact in-memory index + disk cache
  Scanner.cs          parallel scanning, incremental updates, MFT reader
  IndexManager.cs     background indexing, file watchers, rescans
  Search.cs           query parsing, parallel search, ranking, live search
  Network.cs          SMB access, credentials, timeouts, share listing
  Settings.cs         settings, destinations, history, start with Windows
  Shell.cs            icons, thumbnails, open / show / properties, Explorer folder
  UI/                 search window, settings, dialogs, styles, controls
tools/IconGen.cs      icon generator used by build.bat
```

## Data and privacy

Everything stays in the current user's profile, in `%LOCALAPPDATA%\NovaFind\`:

| File | Content |
|---|---|
| `settings.json` | settings and appearance |
| `destinations.json` | search locations. Passwords are stored **only encrypted with Windows DPAPI**. |
| `history.json` | recently opened files and searches |
| `index\*.nfx` | index cache |
| `novafind.log` | diagnostics (never passwords) |

*Settings → Privacy e dati → Cancella tutti i dati locali* deletes all of it and removes the startup entry.

Command line: `--background`, `--toggle`, `--settings`, `--quit`, `--stats`, `--bench <folder> <output> [queries…] [--live]`.

## Known limitations

- Searches file and folder **names**, not file contents.
- Network shares are not watched in real time; they are refreshed on request or on a schedule.
- MFT fast-indexing requires administrator rights; otherwise standard enumeration is used.
- Frosted-glass blur requires Windows 10 1803 or later.
- Private memory committed by .NET/WPF is higher than the working set (~150–200 MB).
