---
name: harmony-hdc-ui-automation
description: Use when working in a HarmonyOS project to inspect or control a real HDC-connected device or emulator from the command line, launch or stop the app, capture screenshots, dump the live UI tree, capture/filter hilog logs for debugging, transfer files to or from the current app sandbox, inject UiTest uiInput actions, design ArkTS UiTest cases, or write/run Hypium Python UI automation with the project's UV/.venv Python environment.
---

# Harmony HDC UI Automation

## Core Rule

Use the project UV environment for all Python commands. Prefer:

```bash
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py doctor
```

If no virtual environment is active, use the repository `.venv` explicitly:

```bash
uv run --python .venv/bin/python python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py doctor
```

Inside the Codex sandbox, `uv run` may fail on `~/.cache/uv` permissions. If the command is needed for the user request, rerun it with escalation instead of switching to global Python.

## Missing Tool Handling

When a local tool or runtime is missing, do not silently switch to a weaker path. Diagnose first:

```bash
command -v uv
command -v hdc
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py doctor
```

If `uv`, `hdc`, DevEco/Hvigor, UiTest command-line support, or Hypium Python packages are missing, read `references/official-ui-automation-notes.md` and suggest the matching install/setup command. Ask before installing packages, downloading SDK/test artifacts, changing shell startup files, or enabling persistent device test mode.

## Continuous Improvement Rule

Prefer extending `scripts/harmony_hdc_ui.py` over repeating raw HDC commands. If the same command pattern is used three or more times in one task, or if it combines multiple fragile steps such as screenshot plus layout dump, add or reuse a script subcommand before continuing long manual testing.

When a command fails because the syntax, path style, option shape, or sandbox behavior is wrong, update both the script and `references/official-ui-automation-notes.md` before finishing. Record the corrected command and the failure trigger so the next run does not repeat the same mistake.

## Workflow

1. Verify device connectivity before acting.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py devices
   ```

   If multiple devices are listed, pass `--target <connect-key>` to every script command. Do not guess the target when actions have real device side effects.

2. Launch or stop the app with HDC.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py start --bundle <bundleName> --ability EntryAbility --module entry
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py stop --bundle <bundleName>
   ```

3. Inspect the real UI before changing tests.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py screencap --out /tmp/harmony-screen.png
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py dump --out /tmp/harmony-layout.json --bundle <bundleName>
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py snapshot --prefix /tmp/harmony-state --bundle <bundleName> --merge-windows
   ```

   Use screenshots for visual state and `dumpLayout` JSON for robust selectors. Prefer `snapshot` when both are needed. Prefer text, id, type, description, or bundle filters over hard-coded coordinates.

4. Capture hilog when a UI action fails, data looks stale, navigation is unexpected, or native/cloud/model behavior needs attribution.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-clear
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-tail --lines 500 --type app --level D --regex "MNN|Agent|Workflow|Exception|Error" --out /tmp/harmony-hilog.txt
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-tail --lines 300 --type app --pid <pid> --out /tmp/harmony-app-hilog.txt
   ```

   Use hilog with screenshot/layout evidence: first capture or clear the focused log window, perform the UI action, then save filtered logs. Prefer `--out` for large logs so the final response can summarize evidence instead of pasting noisy output. Ask before receiving full `/data/log/hilog` disk logs because they can be large and may contain private data.

5. Pull or update app sandbox files when debugging stateful behavior.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-ls /data/storage/el2/base/haps/entry/files/
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-send mobile_file_example/files/workflows /data/storage/el2/base/haps/entry/files/
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-recv /data/storage/el2/base/haps/entry/files/workflows/ mobile_file_example/files/workflows/
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-recv /data/storage/el2/base/haps/entry/files/config.json ~/Desktop/config.json
   ```

   Default bundle is `com.example.mnnllmchat`. Pass `--bundle <bundleName>` for another app and `--target <connect-key>` when multiple devices are connected. Ask before overwriting device files or pulling large/private data.

6. Use command-line UI control only for exploration or simple smoke checks.

   ```bash
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py click 540 1800
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py swipe 540 1900 540 600
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py text "hello"
   uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py key BACK
   ```

   Ask before injecting actions that could delete data, send messages, purchase items, grant permissions broadly, or affect another user's account.

7. For repeatable UI assertions, write ArkTS UiTest cases under `entry/src/ohosTest/ets/test/` and run them with the project Hvigor test flow. Use `$harmony-hvigor-unit-test` when compiling/running those tests.

8. For host-side Python automation, write Hypium Python cases with `TestCase`, `UiDriver`, and `BY` selectors, then run them through the project UV environment. Keep Python dependencies installed in the project `.venv`, not in global Python.

## Official Notes

Read `references/official-ui-automation-notes.md` when writing or modifying test cases, when explaining official API behavior, or when choosing between HDC command-line control, ArkTS UiTest, and Hypium Python.

## Practical Defaults

- Use `hdc list targets` before acting; use `hdc -t <SN>` when more than one device is connected.
- Start the app with `aa start -a <abilityName> -b <bundleName> -m <moduleName>`.
- Stop the app with `aa force-stop <bundleName>`.
- Capture screenshots with `uitest screenCap -p /data/local/tmp/<file>.png`, then pull with `hdc file recv`.
- Dump the UI tree with `uitest dumpLayout -p /data/local/tmp/<file>.json`.
- Use `snapshot --prefix <path>` when a task repeatedly needs both screenshot and UI tree.
- Use `hilog-clear` before focused reproductions and `hilog-tail --out <file>` after the action to capture debuggable evidence.
- Transfer app files with `hdc file send/recv -b <bundleName>`; use `hdc shell -b <bundleName>` for `ls`, `mkdir`, and other sandbox inspection.
- For directory pulls, a remote path ending with `/` means pull the directory contents into the local destination directory.
- Inject exploratory actions with `uitest uiInput`, but convert important flows into ArkTS UiTest or Hypium Python tests.
