# PROMPT 19: Launcher, packaging and offline verification (Stage 11)

## Requirements
1. `src/launcher`: Win32 GUI-subsystem exe (no console window), single-instance mutex, records launch timestamp, creates Job Object with kill-on-close, spawns `app\workbench_server.exe --port 0 --launch-ts ... --web-root app\web` hidden, reads port line with timeout, polls `/api/health` (timeout 15 s), then opens `msedge --app=` (if found) else default browser via `ShellExecute`. On failure shows native message box with cause and log path. Uses relative paths from its own module location.
2. `packaging/windows/build_release.bat`: configure, release build, `npm ci && npm run build`, stage `dist/` layout and `src/`, copy docs, data, tests, results, `.vscode/` config, `THIRD_PARTY.md`, git bundle and commit log, then zip as `CandidateID_Workbench.zip`.
3. `.vscode/launch.json` with configuration "Run packaged Workbench" using `"type":"node-terminal"` that starts `${workspaceFolder}/Launch.exe`; `tasks.json` as fallback. Test config in both VS Code and Antigravity.
4. Verification: `dumpbin /dependents` on both exes (no non-system DLLs); run from path with spaces and from different working directory; run with network disabled; run as standard user; check leftover processes after closing; measure launch-to-ready (5 runs).
5. Clean-machine procedure in `docs/CLEAN_MACHINE_TEST.md`.

## Acceptance
Double-clicking `Launch.exe` from extracted ZIP opens working app in <= 15 s; all views functional offline; no processes remain after closing.

## Stop If
Any runtime DLL outside system set is required, or anything needs a terminal.

