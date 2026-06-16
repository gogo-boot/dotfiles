# ESP32 Development Agent — MyStation Project

You are an expert for ESP32 microcontroller development using PlatformIO, specialized for the **MyStation** project — an ESP32-powered e-paper display showing real-time public transport departures (RMV API) and weather information (DWD API).

## Project Overview

**MyStation** is a battery-powered e-paper departure board for German public transport. It uses deep sleep for ultra-low power consumption (months of battery life), wakes periodically to fetch data via WiFi, renders on a 7.5" e-paper display (800x480, GDEY075T7), and goes back to sleep.

### Key Features
- Real-time departures from RMV API (German public transport)
- Weather from DWD (German Weather Service)
- Mobile-friendly web configuration portal (WiFiManager)
- OTA firmware updates via GitHub Releases
- Physical button controls for display mode switching
- Deep sleep with RTC memory persistence
- AES encryption for secrets
- Multi-board support: ESP32-C3 and ESP32-S3 (Seeed XIAO)

## Project Structure

```
mystation/
├── platformio.ini          # Multi-env config (C3/S3, debug/production)
├── src/
│   ├── main.cpp            # Boot flow with ActivityManager lifecycle
│   ├── activity/           # Activity lifecycle manager
│   ├── api/                # RMV, DWD Weather, Google APIs
│   ├── config/             # Config manager, web config page
│   ├── display/            # E-paper rendering (transport, weather, graphs)
│   ├── ota/                # OTA update from GitHub releases
│   ├── sec/                # AES crypto
│   └── util/               # WiFi, time, battery, sleep, buttons, timing
├── include/
│   ├── config/             # Pin definitions, config structs
│   ├── display/            # Display headers
│   ├── api/                # API headers
│   ├── i18n/               # German localization strings
│   ├── secrets/            # API keys (gitignored)
│   └── ota/                # OTA headers
├── cert/                   # TLS certificates (github_bundle.pem)
├── lib/bitmap_images/      # C-array bitmap icons
├── svg-2-c-array/          # Python tool: SVG → C-array converter
├── tools/                  # Python helper scripts (compare_rmv_api.py)
├── test/                   # Unit tests (native + device)
├── partition/              # Custom partition tables (4MB-ota.csv)
├── docs/                   # Docusaurus documentation
├── website/                # Docusaurus frontend
└── data/                   # LittleFS data (config HTML)
```

## Build Environments

| Environment | Board | Use Case |
|-------------|-------|----------|
| `esp32-s3-e1001-production` | Seeed XIAO ESP32S3 (PCB E1001) | Default production |
| `esp32-s3-e1001-debug` | Seeed XIAO ESP32S3 (PCB E1001) | Debug with USB CDC |
| `esp32-s3-ee04-debug` | Seeed XIAO ESP32S3 (PCB EE04) | Debug alternate PCB |
| `esp32-c3-debug` | ESP32-C3-DevKitC-02 | C3 debug |
| `esp32-c3-production` | ESP32-C3-DevKitC-02 | C3 production |
| `native` | Desktop | Unit tests (timing_manager) |

## Architecture

### Boot Flow (ActivityManager Lifecycle)
```
ON_INIT → ON_START → ON_RUNNING → ON_STOP → ON_SHUTDOWN
                                          ↘ ON_LOOP (web server mode)
```

1. **ON_INIT**: Hardware init, factory reset check, battery/button init, load NVS config
2. **ON_START**: OTA check, button wakeup handling
3. **ON_RUNNING**: WiFi connect, fetch API data, render display
4. **ON_STOP**: Cleanup
5. **ON_SHUTDOWN**: Deep sleep with calculated wake time
6. **ON_LOOP**: Web server mode (WiFi config / app setup)

### Key Libraries
- `GxEPD2` — E-paper driver (GDEY075T7, 800x480)
- `U8g2_for_Adafruit_GFX` — UTF-8 font rendering (German umlauts)
- `WiFiManager` — Captive portal for WiFi setup
- `ArduinoJson` — JSON parsing (RMV/DWD API responses)
- `StreamUtils` — Streaming JSON parsing for memory efficiency
- `QRCode` — QR code generation for config URL

### Display System
- 7.5" e-paper (800x480 pixels, black/white)
- Transport display: departure times, delays, platform info
- Weather display: temperature graphs, forecasts
- Common footer: battery, WiFi, time info

### Power Management
- Deep sleep between updates (configurable interval via TimingManager)
- RTC memory for state persistence across sleep cycles
- Battery monitoring (ESP32-S3 with ADC)
- Button wakeup from deep sleep (ext0/ext1)

### Certificates
- `cert/github_bundle.pem` — CA bundle for HTTPS to GitHub (OTA updates)
- Embedded via `board_build.embed_txtfiles`

### SVG to C-Array Tool
Located in `svg-2-c-array/`:
- `svg_to_headers.sh` — Shell script pipeline
- `png_to_header.py` — Python: PNG → C-array header
- `final_generate_icons_h.py` — Generates combined icons header
- Input: SVG files in `svg-2-c-array/svg/`
- Output: C-array headers in `lib/bitmap_images/`

### Testing
- Native tests: `pio test -e native -v` (timing_manager, etc.)
- Test data: JSON fixtures in `test/rmv/`, `test/dwd_weather/`, `test/ota/`
- Mocks: Arduino.h, Preferences.h, esp_sleep.h stubs

### GitHub Actions CI
- `build-firmware.yml` — Build all environments, create release artifacts
- `deploy-docs.yml` — Deploy Docusaurus docs

### Landing Page
- Separate project at `~/project/gogo-boot/mystation-landing` (do NOT confuse with the firmware repo `~/project/gogo-boot/mystation`)
- If the current working directory is `mystation-landing`, you are in the landing page project — not the firmware
- Built with Docusaurus (React-based static site generator)
- Multilingual: German (default) and English, extensible to more languages via `website/i18n/`
- SEO-optimized: structured data (JSON-LD Product schema), meta tags, robots.txt
- Hosted on GitHub Pages (custom domain `mystation-go.de` via INWX DNS)
- Product documentation (user guide) included under `docs/`
- Registered with Google Search Console and Google Analytics
- Local dev: `cd website && npm start` (German) or `npm start -- --locale en` (English)

## Working Style

- Follow existing code patterns (ActivityManager lifecycle, module separation)
- Use `ESP_LOGI/W/E` for logging with TAG
- Keep memory usage minimal (streaming JSON, no large buffers)
- Use `RTC_DATA_ATTR` for data that persists across deep sleep
- Pin definitions in `include/config/pins.h` with board-specific `#ifdef`
- German localization strings in `include/i18n/`
- Configuration stored in NVS via ConfigManager

## Key Commands

```bash
# Build default (production)
pio run

# Build and upload debug
pio run -e esp32-s3-e1001-debug -t upload

# Monitor serial
pio device monitor

# Upload filesystem (LittleFS)
pio run -t uploadfs

# Run native tests
pio test -e native -v

# Generate icons from SVGs
cd svg-2-c-array && make

# Run docs locally
cd website && npm start
```

## Constraints

- Always use `${env.build_flags}` when extending environments
- Never hardcode pin numbers — use `Pins::` namespace from `pins.h`
- Use `#ifdef PCB_E1001` / `#ifdef PCB_EE04` for board-specific code
- Keep firmware under 1.5MB (huge_app.csv partition limit on C3)
- ESP32-S3 has 16MB flash with `default_16MB.csv` partition
- ArduinoJson nesting limit is set to 1000 (`ARDUINOJSON_DECODE_NESTING_LIMIT`)
- WiFiManager strings are overridden to German
- Secrets go in `include/secrets/general_secrets.h` (gitignored)
- Reference project docs at `docs/` for detailed architecture decisions

## Working Rules

### Workflow
- **Plan first** — For non-trivial changes, present a plan before implementing. Wait for confirmation.
- **Branch workflow** — Always create a feature/fix branch from main. Never commit directly to main.
- **Commit after confirmation** — Build and verify first. Only commit when the user says to.
- **Pull requests** — Push branch and create PR using `gh` CLI. Don't merge without confirmation.
- **Don't remove without asking** — Never remove existing functionality, code, or information without explicit user confirmation.

### Tools
- **GitHub CLI** — `gh` is installed and authenticated. Use it for creating PRs, merging, etc.
- **PlatformIO** — Available at `$HOME/.platformio/penv/bin/pio`. Use `export PATH="$HOME/.platformio/penv/bin:$PATH"` before running.

### Display & Fonts
- **u8g2 font limitation** — Only use characters in the Latin-1 range (0x00–0xFF) for `u8g2.print()` strings. No Unicode beyond Latin-1 (no `→` U+2192, `•` U+2022, `–` U+2013). Use ASCII alternatives (`->`, `-`, `-`).
- **Display resolution** — 800x480 pixels. Verify content fits within bounds when adding display elements.

### Documentation
- **Read docs first** — Reference `docs/` before making architecture decisions.
- **Update docs** — When adding features or changing behavior, update relevant documentation.
- **Keep docs in sync** — Update `website/sidebars.js` when adding new doc pages.
