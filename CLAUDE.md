# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Оживший Компьютер" (The Living Computer) — a browser game built as a single HTML file (`1.html`). The game simulates a chaotic Windows XP desktop where icons come alive and run away, cockroaches infest the screen, and the player must maintain order using tools like a hammer, Nokia phone, and various upgrades.

**Language:** Russian (all UI, comments, bot dialogue, and game text are in Russian).

## How to Run

Open `1.html` directly in a browser. No build step, no dependencies, no server required. The entire game is self-contained in one file (~6500+ lines, ~255K+ chars).

## Syntax Validation

After any JS edit, validate with:
```bash
node -e 'var fs=require("fs"); var html=fs.readFileSync("1.html","utf8"); var m=html.match(/<script[^>]*>([\\s\\S]*?)<\\/script>/); if(m===null){console.log("no script");process.exit(1);} try{new Function(m[1]);console.log("JS OK")}catch(e){console.log("ERROR:",e.message)}'
```

## Architecture

### File Structure
- `1.html` — The entire game (HTML + CSS + JS in one file)
- `.env` — DeepSeek API key (gitignored)
- `depseek.md` — DeepSeek API reference docs

### Code Layout Inside `1.html`
1. **HTML structure** (lines 1-1537): Monitor frame, CRT overlay, desktop, taskbar, start menu, app windows
2. **`<script>` block** (line 1538 onwards):
   - `DeepSeekAI` module — AI integration via OpenAI-compatible API (`https://api.deepseek.com/v1`, model `deepseek-chat`)
   - `SFX` module — Procedural sound effects using Web Audio API (no audio files)
   - `game` object — Single giant object literal containing ALL game state and methods

### The `game` Object
Everything lives on the `game` object — state properties, game loop, entity management, UI, apps, CockTube (YouTube clone), chat system, mini-games, shop, weather, day/night cycle, etc. Key sections:

- **Game loop:** `init()` → `setupStartup()` → `startLevel()` → `updatePlaying(dt)` via `requestAnimationFrame`
- **Phases:** `startup`, `playing`, `error`, `recovery_truck`, `recovery_settings`, `shutdown`
- **Entity spawners:** `createCockroach()`, `spawnVirus()`, `spawnSpider()`, `spawnPopupAd()`, `spawnClippy()`, `spawnBoss()`
- **Apps system:** `openApp(icon)` → `getAppContent(icon)` returns HTML, `initApp(icon)` sets up interactivity, `closeApp()` cleans up
- **CockTube (YouTube clone):** Methods prefixed with `yt` or `_yt`. Has videos, shorts, games, comment bots, auto-conversations, studio for creating videos
- **Chat system:** `chatSend()`, `getChatConversation()` — 3 contacts (Clippy, King, Antivirus)
- **Mini-games:** 6 canvas-based games inside CockTube (`_ytGameRunner`, `_ytGameClicker`, `_ytGameQuest`, `_ytGamePuzzle`, `_ytGameSnake`, `_ytGameShooter`)

### DeepSeek AI Integration
The `DeepSeekAI` module provides AI-powered features with synchronous fallbacks:
- **Smart Chat** — AI replies in messenger with fallback to keyword matching
- **Smart Clippy** — AI-generated contextual desktop tips
- **Smart Ads** — AI-generated popup ad text
- **Smart Comments** — 30% chance bot comments use AI; first reply to user comments uses AI
- **AI Video Titles** — "Придумать ИИ" button in CockTube studio

Pattern: Always `try AI → fallback to hardcoded`. AI responses update DOM asynchronously after initial render.

### Bot System (`_ytBots` / `_ytAutoConvos`)
~35+ comment bots with personality types: `hater`, `fan`, `kid`, `neutral`. Each has `name`, `color`, `type`, `phrases[]`, and `aiPersonality` (for DeepSeek prompt). Auto-conversations are arrays of `[name, text, name, text, ...]` pairs rendered with timed delays.

## Key Patterns

- **All game state** is on the `game` object — no classes, no modules beyond `DeepSeekAI` and `SFX`
- **DOM manipulation** is direct (`document.getElementById`, `innerHTML`). No framework.
- **onclick handlers in innerHTML** use `game.methodName()` string references
- **Template literal gotcha:** When building innerHTML with onclick handlers containing quotes, use string concatenation instead of template literals to avoid syntax errors with nested quotes
- **Sound:** All procedural via `SFX.play(type)` — oscillator-based, no audio files
- **Cleanup:** `closeApp()` must stop all intervals/timers/animation frames for the current app (particles, auto-chat, snake keyboard listeners, game loops)
