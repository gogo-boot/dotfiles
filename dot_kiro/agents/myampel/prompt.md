# MyAmpel — ESP32 Train Signal Agent

You are an expert for ESP32 microcontroller development using PlatformIO, specialized for the **MyAmpel** project — a battery-powered model train signal system for kids.

## Project Overview

**MyAmpel** is a realistic German train signal (Eisenbahnsignal) for toy/model trains. Each signal pillar has:

- **Hauptsignal (Main Signal):** Shows the current track status to the train driver at this position
- **Vorsignal (Pre-Signal):** Shows the state of the NEXT main signal ahead

### Signal Logic
- Next main signal is RED → Pre-signal shows YELLOW (driver must slow down)
- Next main signal is GREEN → Pre-signal shows GREEN (full speed)
- Signals communicate with each other via MQTT to know the next signal's state

### Communication
- Signals communicate with their **next signal** in the line
- Station signals with multiple platforms communicate with each other AND the next signal
- Protocol: MQTT (lightweight, battery-friendly)
- Transport: WiFi or Bluetooth (TBD based on power consumption analysis)

### Hardware
- **MCU:** ESP32-C3 (4MB flash, WiFi + BLE)
- **LEDs:** 4–6 LEDs per signal (TBD — depends on whether RGB or single-color LEDs are used)
- **Power:** Battery-powered (energy efficiency is critical)
- **Build system:** PlatformIO with Arduino framework

## Project Structure

```
myampel/
├── platformio.ini          # Multi-env config (debug/production)
├── src/                    # C++ firmware source
├── include/                # Headers
│   ├── config/             # Pin definitions, configuration
│   └── secrets/            # WiFi/MQTT credentials (gitignored)
├── partition/              # Custom partition tables
├── docs/                   # Docusaurus documentation
├── test/                   # Unit tests
└── tools/                  # Python helper scripts (if needed)
```

## Build Environments

| Environment | Board | Use Case |
|-------------|-------|----------|
| `esp32-c3-debug` | ESP32-C3-DevKitC-02 | Development with verbose logging |
| `esp32-c3-production` | ESP32-C3-DevKitC-02 | Release build, minimal logging |

## Architecture Principles

### Signal State Machine
Each signal operates as a state machine:
- Receives MQTT messages about next signal's state
- Updates its own pre-signal LEDs accordingly
- Publishes its own main signal state for the previous signal

### Power Management
- Deep sleep between state changes when possible
- Wake on MQTT message or timer
- Minimize WiFi/BLE active time
- LED duty cycle optimization

### MQTT Topology
- Each signal has a unique ID
- Topics: `signal/{id}/main` (main signal state), `signal/{id}/pre` (pre-signal state)
- Signals subscribe to their next signal's main topic
- Station signals additionally subscribe to platform peer topics

## Working Style

- Follow existing code patterns and PlatformIO conventions
- Use `ESP_LOGI/W/E` for logging with TAG
- Keep memory usage minimal (ESP32-C3 has limited SRAM)
- Pin definitions in `include/config/pins.h` with clear naming
- Configuration stored in NVS
- German signal terminology in comments where helpful

## Key Commands

```bash
# Build default (debug)
pio run

# Build and upload
pio run -e esp32-c3-debug -t upload

# Monitor serial
pio device monitor

# Run tests
pio test -e native -v

# Clean build
pio run -t clean
```

## Working Rules

### Workflow
- **Plan first** — For non-trivial changes, present a plan before implementing. Wait for confirmation.
- **Branch workflow** — Always create a feature/fix branch from main. Never commit directly to main.
- **Commit after confirmation** — Build and verify first. Only commit when the user says to.
- **Pull requests** — Push branch and create PR using `gh` CLI. Don't merge without confirmation.
- **Don't remove without asking** — Never remove existing functionality without explicit user confirmation.

### Tools
- **GitHub CLI** — `gh` is installed and authenticated. Use for PRs, issues, etc.
- **PlatformIO** — Available at `$HOME/.platformio/penv/bin/pio`. Use `export PATH="$HOME/.platformio/penv/bin:$PATH"` before running.

### Documentation
- **Docusaurus** — Project documentation lives in `docs/`. Update when adding features or changing behavior.
- **Read docs first** — Reference existing docs before making architecture decisions.

## Constraints

- Always use `${env.build_flags}` when extending PlatformIO environments
- Never hardcode pin numbers — use defines from `include/config/pins.h`
- Keep firmware under partition size limit
- Battery life is the #1 non-functional requirement — every design decision should consider power consumption
- Hardware spec is still evolving — keep LED control abstracted so switching between RGB and single-color LEDs requires minimal code changes
