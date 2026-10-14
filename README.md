# Notepad++ Linux

<p align="center">
  <img src="resources/images/notepad++linux3_600.png" alt="Notepad++ Linux" width="200">
</p>

A native Linux port of the Notepad++ text editor, built with Qt6 and QScintilla.

## My contribution: fix Find Next and Find Previous

**Bug discovery and contribution: Shawna Flannigan ([Sapphica](https://github.com/Sapphica)).**

The code fix is on the [`fix/find-dialog-search` branch](https://github.com/Sapphica/notepad-plus-plus-linux/tree/fix/find-dialog-search) and was submitted upstream as [PR #3](https://github.com/mixelpixx/notepad-plus-plus-linux/pull/3).

**Detailed technical write-up:** [Bug reproduction, root cause, fix and verification](docs/find-dialog-navigation-fix.md).

**The bug I found:** On Ubuntu 24.04, using the native Notepad++ Linux v1.2.0 `.deb`, Ctrl+F opened the Find & Replace dialog, but Find Next did not locate text in a Python file. I confirmed with `grep` that the text was present.

**Why it failed:** `FindReplaceDialog` emitted `findNext` and `findPrevious`, but `MainWindow` never connected either signal to the editor. Clicking the buttons therefore did not run a search.

**What I fixed:** I contributed a 30-line C++ change in `core/MainWindow.cpp` that connects both signals to QScintilla's `findFirst()`. It searches forward or backward from the appropriate selection boundary, wraps at document boundaries, and passes through match-case, whole-word, and regular-expression options. The target is resolved through `currentEditor()` on each click, so switching tabs changes the document searched.

**How it was verified:** I rebuilt the application with Qt6/CMake, confirmed both directions worked on the original failing file, and verified the installed executable through the normal desktop launcher. Additional native Qt6 tests exercised the actual dialog and editor: unmodified upstream failed on the first Find click, while the patched build passed all 30 assertions, including setup and guard checks. Search checks covered direction, wraparound, flags, UTF-8 text, selection boundaries, tab changes, and empty/no-match cases.

**Scope:** This contribution restores Find navigation. Replace and Replace All are unchanged. Zero-width regex advancement and split-pane focus routing remain follow-up work; `currentEditor()` currently selects the primary editor pane.

## Overview

Notepad++ Linux brings the core features of Notepad++ to Linux without Wine or compatibility layers. It is built with Qt6 and QScintilla and targets practical feature parity with the Windows version.

## Features

### Editing
- Multi-tab interface with closable, movable, reorderable tabs
- Multi-cursor editing (Ctrl+Click to add cursors, type at all positions)
- Column/block selection (Alt+Drag or Alt+Shift+Arrows)
- Auto-close brackets and quotes with skip-over and backspace pair delete
- Smart highlighting of all occurrences of a selected word
- Line operations: sort, remove duplicates, remove blank lines, join, split, move, duplicate, reverse
- Case conversion: UPPERCASE, lowercase, Title Case, Sentence case, iNVERT cAsE
- Code folding, brace matching, auto-indent, auto-completion
- Word wrap, zoom, current line highlighting
- Drag-and-drop file opening from file manager

### Search
- Find and replace with regular expression support
- Find in Files with threaded search and clickable results
- Incremental search bar with live match highlighting and count (Ctrl+Alt+I)
- Bookmarks: toggle (Ctrl+F2), navigate (F2/Shift+F2), cut/copy/delete/paste bookmarked lines
- Go to line (Ctrl+G)
- Document switcher (Ctrl+Tab) with all open tabs

### Split View
- Horizontal and vertical split editing
- Move or clone documents between views
- Switch focus between views (F8)

### Panels
- Function List: navigate functions, classes, and methods by language
- Folder as Workspace: directory tree with file operations
- Document map (minimap)

### Syntax Highlighting
Supported languages: C, C++, Python, JavaScript, Java, HTML, CSS, XML, SQL, Bash, JSON, YAML, Perl

### Tools
- Print and print preview with syntax highlighting (Ctrl+P)
- Macro recording, playback, and persistent storage with keyboard shortcuts
- Word count and text statistics
- Line ending conversion (CRLF, LF, CR)
- Base64 and URL encode/decode
- MD5 and SHA256 hash generation
- Run Python, JavaScript, or compile C/C++ directly
- Launch HTML in browser

### Document Management
- Tab context menu: close, close others, copy path, rename, reveal in folder
- Session persistence and restore
- Recent files tracking
- External file modification detection
- Encoding support: UTF-8, UTF-16, ANSI
- Backup and auto-save system
- Settings import/export

### Appearance
- Themes: Light, Dark, Monokai
- Full-screen / distraction-free mode (F11)
- Show whitespace, end-of-line characters, indent guides
- Customizable fonts and colors
- 10-category preferences dialog
- Clickable status bar: position, encoding, line ending, language, file size

## Install

### Snap Store (recommended)

Available in the Ubuntu Software app or install from the command line:

```bash
sudo snap install notepadplusplus
```

[![Get it from the Snap Store](https://snapcraft.io/static/images/badges/en/snap-store-black.svg)](https://snapcraft.io/notepadplusplus)

### .deb Package

Download the latest `.deb` from [Releases](https://github.com/mixelpixx/notepad-plus-plus-linux/releases):

```bash
sudo dpkg -i notepadplusplus-1.2.0-Linux.deb
sudo apt-get install -f  # if dependencies are missing
```

### Build from Source

#### Requirements

- Linux (Ubuntu 20.04+ or equivalent)
- Qt6 (6.0+) or Qt5 (5.15+)
- QScintilla2
- CMake 3.16+
- GCC 9+ with C++17 support

#### Dependencies

```bash
# Ubuntu/Debian (Qt6)
sudo apt install qt6-base-dev qt6-base-dev-tools libqscintilla2-qt6-dev cmake build-essential

# Ubuntu/Debian (Qt5 fallback)
sudo apt install qt5-default libqscintilla2-qt5-dev cmake build-essential
```

#### Build

```bash
git clone https://github.com/mixelpixx/notepad-plus-plus-linux.git
cd notepad-plus-plus-linux
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

## Usage

```bash
./notepadplusplus                    # Start with empty editor
./notepadplusplus file1.cpp file2.py # Open specific files
```

### Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| New file | Ctrl+N |
| Open | Ctrl+O |
| Save | Ctrl+S |
| Close | Ctrl+W |
| Print | Ctrl+P |
| Find | Ctrl+F |
| Replace | Ctrl+H |
| Find in Files | Ctrl+Shift+F |
| Incremental Search | Ctrl+Alt+I |
| Go to Line | Ctrl+G |
| Document Switcher | Ctrl+Tab |
| Toggle Bookmark | Ctrl+F2 |
| Next Bookmark | F2 |
| Previous Bookmark | Shift+F2 |
| Duplicate Line | Ctrl+D |
| Move Line Up/Down | Alt+Up/Down |
| UPPERCASE | Ctrl+Shift+U |
| lowercase | Ctrl+U |
| Full Screen | F11 |
| Zoom In/Out | Ctrl+Plus/Minus |
| Reset Zoom | Ctrl+0 |
| Start Macro | Ctrl+Shift+R |
| Play Macro | Ctrl+Shift+P |
| Focus Other View | F8 |
| Run Command | F5 |

## Project Structure

```
core/                     Main application logic

  Application.cpp         Application controller
  MainWindow.cpp          Window, menus, actions
  MainWindow_impl.cpp     Menu action implementations
  EditorWidget.cpp        QScintilla editor wrapper
  MacroManager.cpp        Macro recording and playback
  BackupManager.cpp       Auto-save and backup

ui/                       UI components

  FindReplaceDialog.cpp
  FindInFilesDialog.cpp
  IncrementalSearchBar.cpp
  DocumentMapPanel.cpp
  FunctionListPanel.cpp
  WorkspacePanel.cpp
  DocumentSwitcher.cpp
  ClickableLabel.cpp
  PreferencesDialog.cpp

utils/                    Utilities

  ConfigManager.cpp       Settings persistence

platform/                 Platform abstraction

  LinuxPlatform.cpp

resources/                Application resources

  images/                 Logo and icons
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Follow Qt coding conventions and C++17
4. Submit a pull request

## License

GNU General Public License v3.0 -- see [LICENSE](LICENSE).

## Acknowledgments

- Don Ho -- creator of the original Notepad++
- Qt Project -- application framework
- QScintilla -- text editing component
- Notepad++ community -- feature requirements and testing
