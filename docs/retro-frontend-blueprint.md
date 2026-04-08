# Retro Emulator Front-End Blueprint

## Product goals

Build a cross-platform emulator front end that can outperform LaunchBox, RetroBat, and EmulationStation in:

- startup speed
- library indexing speed
- metadata quality and reliability
- flexibility for themes/layouts
- emulator compatibility and per-game tuning

## Target platforms

- Windows (x64/ARM)
- Linux desktop + handheld distros
- macOS
- Optional later targets: Android, Steam Deck first-class profile, web companion

## Technical architecture (recommended)

### 1) App shell

Use a **Rust + Tauri** desktop shell with a TypeScript UI (Angular/React).

Why:
- native-level performance for filesystem scanning and indexing
- secure desktop APIs
- easy packaging across Windows/Linux/macOS
- good fit for emulator process management and IPC

### 2) Core services

- **Library service**: ROM discovery, hashing, dedupe, metadata linking, playlists
- **Emulator orchestrator**: launches RetroArch cores and standalone emulators with profiles
- **Asset service**: pulls box art/screenshots/video from configured providers and caches locally
- **Theme/layout service**: dynamically loads skins, templates, icon packs, and view presets
- **Achievements service**: RetroAchievements auth, game mapping, runtime event updates
- **Sync service (optional)**: profile/cloud sync for metadata and config

### 3) Data layer

Use SQLite (WAL mode) with a local cache folder.

Key tables:
- `games`
- `rom_files`
- `systems`
- `emulators`
- `launch_profiles`
- `assets`
- `tags`
- `collections`
- `play_history`
- `achievements`

Indexes:
- crc/sha1/md5 on ROMs
- path and modified time
- system/platform keys
- full-text index on title, alt title, publisher, genre

## Emulator support model

### RetroArch core integration

- per-system default core
- per-game core override
- per-game RetroArch argument templates
- save-state and shader presets surfaced in UI

### Standalone emulator integration

Support a registry-driven adapter model:

- adapter = executable detection + CLI template + save path mapping + exit/health behavior
- examples: PCSX2, Dolphin, RPCS3, Cemu, Xemu, Yuzu forks where legal and available

### Smart launcher

At launch time:
1. identify ROM + system
2. resolve best profile (user override > benchmark profile > default)
3. build command
4. inject runtime options (fullscreen, bezel, controller profile, achievements flags)
5. track process and update play-session telemetry

## ROM scanner and library management

## Scanner pipeline

1. crawl configured roots (parallel workers)
2. extension/system inference
3. archive handling (zip/7z/chd cues where relevant)
4. hash computation (fast hash + canonical hash)
5. duplicate resolution and merge
6. metadata match and confidence scoring
7. asset pull queue

## Library features

- one-click import wizard
- smart collections (genre/year/developer/custom rules)
- custom tags and hidden titles
- favorites + recently played + never played
- merge regional variants under a primary title
- parental and content visibility filters
- bulk editor for metadata and emulator profile assignment

## Themes, layouts, and UX performance

### Theming

- CSS variable token system + JSON layout descriptors
- theme packs with inheritance
- live theme reload without app restart

### Layout presets

- grid wall, wheel/carousel, details-first, handheld-friendly, kiosk mode
- per-system and per-collection layout override

### Performance tactics

- virtualized lists/grids
- texture/video thumbnail prefetch windows
- incremental rendering and memoized filters
- asset cache tiers (RAM + disk)

Target UX metrics:
- cold boot to interactive: < 2s on mid-tier SSD machine
- open 10k-game library view: < 150ms input latency
- search response: < 50ms for cached index queries

## RetroAchievements integration

- account linking + token handling
- game ID matching using hash/signature mappings
- live status panel (unlocked, hardcore state, rich presence)
- per-game achievement progress + session notifications
- graceful fallback when unsupported game/system

## Asset providers and scraping reliability

Primary sources:
- EmuMovies (if user has credentials)
- plus other user-configured sources with reliability ranking and retry policies

Rules:
- source priority + merge strategy
- local checksum to prevent repeated downloads
- legal/terms-aware provider abstraction
- manual override lock (don’t auto-overwrite user-chosen artwork)

## "All systems + arcade" strategy

- system definition registry (JSON)
- arcade split profiles (MAME/FBN variants)
- per-system file-extension and launch heuristics
- import packs for known BIOS requirements and validation checks

## Benchmarking to beat incumbents

Create repeatable benchmark scenarios:

- 1k / 10k / 50k ROM import timing
- metadata hit rate and mismatch rate
- launch success rate by platform/system
- UI frame pacing on low-end iGPU hardware
- memory footprint during long browsing sessions

Publish a benchmark dashboard inside dev builds so regressions are visible before release.

## Security and stability

- sandboxed command templates for emulator launch
- strict path validation
- crash-safe writes for metadata and profiles
- background task watchdog + auto-retry
- diagnostics bundle export for support

## Implementation roadmap (phased)

### Phase 1: foundation (4–6 weeks)
- Tauri shell + UI scaffold
- settings model, system registry, emulator adapter interface
- initial library scan + SQLite schema
- basic game launch (RetroArch + one standalone emulator)

### Phase 2: library depth (4–8 weeks)
- high-performance scanner + hash pipeline
- metadata matching and manual correction UI
- collection manager and bulk edit tools
- game details pages + history tracking

### Phase 3: polish and extensibility (4–8 weeks)
- theme engine and layout presets
- achievements integration
- asset ingestion and video backgrounds
- profiling + startup optimizations

### Phase 4: competitive hardening (ongoing)
- cross-platform QA matrix
- benchmark automation and optimization passes
- plugin API for community extensions
- installer/updater and migration tooling

## Suggested immediate next tasks for this repository

1. Add a desktop host layer plan (Tauri) while preserving existing web component support.
2. Define a normalized schema for ROMs, games, assets, and emulator profiles.
3. Implement a first-pass scanner service with hash + extension inference.
4. Add an emulator adapter abstraction to launch RetroArch and one standalone emulator.
5. Add a metadata provider interface with mock/local provider first, then real providers.

## Success criteria (v1)

- Import 10,000 ROM files in under 10 minutes on a mainstream SSD machine.
- Maintain 60 FPS navigation in primary views with 10,000+ entries loaded.
- Achieve >95% automated metadata match accuracy for mainstream console sets.
- Launch success rate >99% for tested systems/profiles.
- Theme/layout switching in under 300ms with no restart.
