# MyStation Project Rules

## Git Rules (STRICT)

- **NEVER commit or push directly to main.** Always create a feature/fix branch first.
- Every change must go through a Pull Request — no exceptions.
- Only push to feature branches. The user merges to main via PR.
- Always confirm `git branch --show-current` before committing.

## Two Separate Repos (do not confuse)

- **Firmware:** `~/project/gogo-boot/mystation` — ESP32 code + developer/maintainer docs.
- **Landing / user guide:** `~/project/gogo-boot/mystation-landing` — marketing site + the
  end-user manual.
- The **user manual belongs ONLY in the landing repo.** Developer docs belong ONLY in the
  firmware repo. Do not put user-facing how-to content in the firmware repo.

## Bilingual Docusaurus Docs (both repos)

- English source lives in `docs/`. German lives in
  `website/i18n/de/docusaurus-plugin-content-docs/current/`.
- **Always mirror documentation edits in BOTH the EN and DE files.**
- The sidebar is a **single shared config** (`website/sidebars.js`) — it is NOT per-locale, so
  a reorder/add affects both languages at once (no separate DE sidebar edit needed).
- Verify docs changes with `cd website && npm run build` — expect `[SUCCESS]` for **both**
  locales (e.g. `build/` and `build/en`, or `build/` and `build/de`).
- The landing repo's commit hook warns about a missing Jira (`IWC-xxx`) ref — **harmless for
  this project, ignore it.**
- For large section moves within a doc, prefer a small scripted move over a giant find/replace,
  then verify structure + build.

## Firmware Build & Test

- Add PlatformIO to PATH first: `export PATH="$HOME/.platformio/penv/bin:$PATH"`.
- Debug build env: `esp32-s3-e1001-debug`. Native unit tests: `pio test -e native`
  (suites use per-suite `_impl.cpp` forwarders; e.g. `test_solar_math`, `test_timing_manager`).
- Icon pipeline lives in `svg-2-c-array/` — regenerate with `make` (NEVER `make -j`; Inkscape
  has a parallel-conversion bug).

## Display / Fonts

- **u8g2 renderer is Latin-1 only (0x00–0xFF).** No Unicode beyond Latin-1 (no `→` `•` `–`).
  Use ASCII (`->`, `-`). The `²` superscript is byte `0xB2` (Latin-1 safe).
- Display is 800×480, black/white, no gray. Verify content fits the bounds.

## Icons

- UI glyphs come from **Lucide (ISC)** and **Tabler Icons (MIT)** — both open-source and free
  for commercial use. Weather=Weather Icons (OFL/MIT), Battery=Material Symbols (Apache 2.0),
  WiFi=Phosphor (MIT). Record any new library's license in `svg-2-c-array/README.md`.

## Solar / Day-Browse Feature (shipped)

- The "Sonnenstrom" solar-radiation feature is complete and merged. Day browse has two
  contexts (`BrowseContext { WEATHER, SOLAR }`, RTC-only).
- Button model (Weather-Only mode): from the default today view, **B2 enters weather browse**,
  **B3 enters solar browse**; while browsing **B2 = +1 day, B3 = -1 day** in the current
  context; **B1 exits to today** (the only way to switch context). Solar browse includes
  day 0; weather browse skips it. Build on this rather than reinventing it.
