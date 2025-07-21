# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VimDocs is a Chrome extension that brings Vim keybindings to Google Docs. The extension injects a content script that intercepts keyboard events and translates Vim commands into appropriate Google Docs actions.

## Architecture

### Core Components

- **manifest.json**: Chrome extension manifest that defines permissions and content script injection rules
- **content.js**: Main content script that implements the Vim modal system and keybinding handlers
- **README.md**: Feature checklist and installation instructions

### Key Architecture Patterns

**Modal System**: Uses a Proxy-based state manager (`modeProxy`) that automatically updates the UI when the current mode changes between:
- `NORMAL`: Default Vim command mode
- `INSERT`: Text input mode  
- `VISUAL`: Character-wise visual selection
- `VISUAL_LINE`: Line-wise visual selection

**Event Handling**: Two-tier event system:
1. Main keydown listener that handles mode-specific commands
2. Dynamic event listeners for complex operations (deletion, replacement) that temporarily override the main handler

**Platform Detection**: Cross-platform compatibility with `isMacOS()` detection that adjusts keybindings for macOS vs Windows/Linux keyboard shortcuts.

**Google Docs Integration**: 
- Targets `.docs-texteventtarget-iframe` for event injection
- Uses `simulateKeyPress()` to dispatch synthetic keyboard events
- Handles cursor positioning via `.kix-cursor` and `.kix-cursor-caret` elements

## Development Commands

This is a simple Chrome extension with no build system - development is done by loading the unpacked extension directly in Chrome developer mode.

### Installation for Development
```bash
# No build commands needed - load directly as unpacked extension
# 1. Open chrome://extensions/
# 2. Enable Developer mode
# 3. Click "Load unpacked" and select this directory
```

### Testing
No automated test framework is configured. Testing is done manually in Google Docs by:
1. Loading the extension
2. Opening a Google Docs document  
3. Testing Vim keybindings in different modes

## Key Implementation Details

**State Flags**: Several global flags manage complex operations:
- `replacementPending`: Prevents main handler during 'r' replacement
- `deletionPending`: Prevents main handler during deletion commands
- `insideDeletion`: Prevents recursive deletion handlers

**Cursor Management**: 
- `setCursorWidth()` adjusts cursor appearance based on current mode
- Mode indicator shows current state in bottom-left corner

**Platform-Specific Behavior**:
- macOS uses Cmd+Arrow for line navigation  
- Windows/Linux uses Ctrl+Arrow and Home/End keys
- Different word boundary detection between platforms

## Known Issues & Limitations

Several features have known bugs documented in the code:
- '0' command (start of line) has positioning issues
- 'O' command (open line above) doesn't work at beginning of line
- 'dd' (delete line) is buggy with lists
- Visual mode yank operations need improvement
- 'gg' command interference when used repeatedly

## Extension Scope

The extension only activates on `*://docs.google.com/document/*` URLs as defined in the manifest content script matches.