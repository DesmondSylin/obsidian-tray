# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This repository contains a **compiled Obsidian plugin** with only the built artifacts:
- `main.js` - Complete plugin implementation (495 lines, single file)
- `manifest.json` - Plugin metadata (version 0.3.5)
- `README.md` - User documentation
- `obsidian.png` / `tray.png` - Assets

**Important**: There is no TypeScript source code, no build system, and no package.json. All development happens directly in `main.js`.

## Plugin Architecture

### Core Dependencies
- **Obsidian API** (`require("obsidian")`) - Plugin lifecycle, settings, file management
- **Electron Remote** (`require("electron").remote`) - Window management, tray, global shortcuts
  - `app` - Application lifecycle, startup settings, dock/taskbar control
  - `Tray` / `Menu` - System tray icon and context menu
  - `globalShortcut` - Global keyboard shortcuts
  - `getCurrentWindow` / `BrowserWindow` - Window manipulation

### Main Components

**1. Window Management** (lines 45-98)
- `vaultWindows` Set - Tracks all vault windows
- `maximizedWindows` Set - Preserves maximized state during minimize/restore
- `observeWindows()` - Sets up window lifecycle listeners
- `showWindows()` / `hideWindows()` / `toggleWindows()` - Core window visibility control
- Window close interception (lines 100-120) - Prevents quit when "run in background" enabled

**2. Tray Icon** (lines 184-228)
- `createTrayIcon()` - Builds system tray with context menu
- Platform-specific click behavior (macOS shows menu, others toggle windows)
- Menu actions: Quick Note, Show/Hide, Relaunch, Close Vault
- Custom icon support via base64 data URLs

**3. Global Hotkeys** (lines 230-248)
- `registerHotkeys()` / `unregisterHotkeys()` - Electron globalShortcut registration
- Default: `CmdOrCtrl+Shift+Tab` (toggle windows), `CmdOrCtrl+Shift+Q` (quick note)
- Format: [Electron accelerator strings](https://www.electronjs.org/docs/latest/tutorial/keyboard-shortcuts#accelerators)

**4. Quick Notes** (lines 161-179)
- `addQuickNote()` - Creates note with date-based filename (Moment.js format)
- Forces creation relative to vault root (bypasses "same folder as current file" setting)
- Automatically shows windows and opens note in source mode

**5. Settings** (lines 250-361, 375-443)
- `OPTIONS` array - Declarative settings configuration
- `SettingsTab` class - Renders settings UI with custom input types (toggle, text, moment, hotkey, image)
- Reactive onChange handlers for live updates

**6. Platform-Specific Behavior**
- **macOS**: `app.dock.hide()` / `app.dock.show()` for taskbar icon hiding (lines 62-73, 122-129)
- **macOS**: Tray icon click shows menu instead of toggling windows (lines 222-227)
- **Windows/Linux**: Tray icon click toggles window visibility

**7. Exposed Plugin API** (lines 481-485)
Other plugins can access these methods via the plugin instance:
```javascript
getCurrentWindow  // Returns current BrowserWindow
getWindows        // Returns array of vault windows
showWindows       // Shows all vault windows
hideWindows       // Hides/minimizes all vault windows
toggleWindows     // Toggles window visibility
```

### Plugin Lifecycle

**onload()** (lines 447-476):
1. Load saved settings
2. Register settings tab
3. Create tray icon
4. Register global hotkeys
5. Set launch-on-startup configuration
6. Observe window lifecycle
7. Intercept window close if "run in background" enabled
8. Hide taskbar icons if configured
9. Register commands for command palette

**onunload()** (lines 477-479):
- Calls `cleanup()` to unregister hotkeys, restore taskbar, allow window close, destroy tray

## Development Guidelines

### Making Changes
1. Edit `main.js` directly - this is the source of truth
2. Test in Obsidian by:
   - Copying plugin to `.obsidian/plugins/tray/`
   - Reloading Obsidian or toggling plugin off/on
3. Update `manifest.json` version when releasing
4. Commit changes to the repository

### Key Patterns to Preserve
- **Logging**: All operations log with `log(LOG_PREFIX: message)` for debugging
- **Platform detection**: Check `process.platform === "darwin"` for macOS-specific code
- **Settings reactivity**: Use `onChange` callbacks in OPTIONS array for live updates
- **Error handling**: Global hotkey operations wrapped in try-catch (silent failures)
- **Window state preservation**: Track maximized windows separately to restore correctly

### Common Pitfalls
- **macOS dock hiding**: Affects ALL vaults, requires careful focus listener handling (lines 62-73)
- **Window close interception**: Must prevent both renderer (`beforeunload`) and main (`close`) events
- **Quick note creation**: Must force vault root as parent, not active file location (line 174)
- **Hotkey format**: Must use Electron accelerator syntax, not Obsidian hotkey format

### Debugging
- Check browser console for `obsidian-tray:` prefixed logs
- Verify global shortcuts with `globalShortcut.isRegistered(accelerator)`
- Inspect tray icon with `tray.isDestroyed()`

## Git Workflow

This repository tracks compiled artifacts, not source code. Standard workflow:
1. Make changes to `main.js`
2. Test in Obsidian
3. Commit with descriptive messages
4. Push to appropriate branch (follows `claude/*` naming for automated workflows)
