# Working notes: Backtale Companion

Running notes for the working session. Not user-facing; kept short and updated after each milestone.

## Status

| Step | State |
|---|---|
| Read code and the three branches | done |
| Handoff from the lead (branches finished) | waiting |
| Merge the three branches into one working branch | not started |
| Green downloadable macOS build, new UI, in-app capture | not started |
| Backtale connection (settings, start/stop live meeting, chunk streaming, buffering) | not started |
| "Questions to ask" overlay (hidden from screen sharing) | not started |
| Windows and Linux parity | not started |
| Signing and notarization | needs the owner's Apple Developer decision |

## What exists

Upstream (main) is Andre Saunite's transcriber, unchanged: a Tauri shell (`src-tauri/`, Rust) that launches a frozen Python engine
(`transcriber.py`, `transcription_engine.py`, faster-whisper) on demand. The window is plain HTML/CSS/JS in `src/` with no build step.
Rust commands in `src-tauri/src/sidecar.rs` start and stop live sessions and file transcriptions and list devices; events
`transcript-line`, `sidecar-log` and `file-transcription-complete` are the contract with the front end.
Capture backends: WASAPI (Windows), PulseAudio/PipeWire monitor (Linux), and on macOS a Swift Core Audio process-tap helper
(`macos/audiotap-helper`, macOS 14.4+) driven by `macos_capture.py`. Specs and past changes live in `openspec/`.

The three branches the lead is finishing (read only until handed over):

- `claude/macos-build` (d03e83b): `.github/workflows/macos-build.yml` builds an Apple Silicon `.dmg`, a zipped `.app` and a command-line
  archive on every push and uploads them as run artifacts for 14 days. Also `docs/BACKTALE-FORK.md` and `docs/macos-test-checklist.md`.
- `claude/ui-redesign` (3efa765): new simple Backtale-looking window (`src/index.html`, `main.js`, `style.css`, new `ui.js` and
  `questions.js`), tokens in `docs/design-tokens.md`, upstream design docs moved to `docs/upstream/`, a hidden questions placeholder,
  updated GUI tests and `tests/ui_screenshots.py`.
- `claude/macos-capture` (24fe59b): in-app macOS live listening (`src-tauri/src/macos.rs`, `support.rs`, `src/macos-live.js`),
  entitlements, permission texts, "Test audio capture" and a diagnostic report, plus new tests.

Expected merge friction: `src/index.html`, `src/main.js`, `tests/test_gui.py`, `src-tauri/src/sidecar.rs` and
`docs/macos-test-checklist.md` are touched by more than one branch. `macos-build` is already inside the other two.

## How to build

- macOS (the only build the owner uses): push a branch, open the GitHub Actions run named `macos-build`, download the `macos`
  artifact. Builds are ad-hoc signed and not notarized, so macOS asks to allow the app on first open.
- Linux: see `docs/building.md` (system libraries, `cargo tauri build`, frozen engine from a venv).

## How to test without a Mac

- `python run_tests.py` runs `cargo test` plus every root and `tests/` suite. Here the Python suites need extra packages that are
  not installed in this environment (`av`, `faster-whisper`, Python `playwright`); install only what a given suite needs, since disk
  is tight. `cargo test` and the node-based UI checks run as-is (node and Chromium with Playwright are preinstalled).
- UI: the front end runs in a browser against `tests/fake_tauri.js`, so layout, wording and states can be checked and screenshotted
  on Linux. The macOS-only parts (permission prompts, the audio tap, signing) cannot be verified here.

## The owner-on-Mac loop

1. A change is pushed; the build runs on GitHub; I check the run and fix failures from the logs.
2. The owner downloads the artifact, opens the app on his Mac and follows the plain-language test checklist.
3. Anything unclear is observable in the app: the "Test audio capture" screen and "Copy diagnostic report" button.
   The owner pastes the report back to the lead, who passes it to me.
4. I fix and push; repeat.

Rules that shape every change: nothing the user sees mentions a terminal, commands, scripts, sidecar or Python; the look follows
Backtale; the repo is public, so no Backtale server code, plans, secrets or customer data; keep Andre's notices and history.

## Open questions

- Backtale side: the exact request format for chunks and for the "questions to ask" feed (the API doc on `claude/live-ingest` in
  the private repo is the source; ask the lead for access or a summary when I start the connection work).
- Where the device token is stored on the Mac (Keychain is the natural place).
- Hiding the overlay from screen sharing on macOS (window sharing type) needs a real-Mac check through the diagnostic report.
- Signing and notarization: waiting on the owner's Apple Developer account decision.
- Whether to keep upstream's release workflows as they are (currently untouched).
