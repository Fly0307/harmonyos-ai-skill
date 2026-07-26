# Official UI Automation Notes

## Sources Consulted

- Huawei HarmonyOS Hypium Python guidelines: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hypium-python-guidelines
- Huawei HarmonyOS UiTest guidelines: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/uitest-guidelines
- Huawei HDC / AA tool guidance surfaced through Context7 official HarmonyOS Guides.
- Huawei HarmonyOS hilog guide: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hilog
- Huawei HDC sandbox file transfer guidance surfaced through Context7 official HarmonyOS Guides.
- uv documentation surfaced through Context7 official `astral-sh/uv` docs.

## Capability Split

Use three layers deliberately:

- **HDC command-line control:** fast device inspection, screenshots, layout dumps, launch/stop, and simple click/swipe/text/key smoke checks.
- **ArkTS UiTest:** repeatable in-project UI assertions using `@kit.TestKit` / `@ohos/hypium`, `Driver`, `ON`, `abilityDelegatorRegistry`, and `expect`.
- **Hypium Python:** host-side device automation using DevEco Testing Hypium Python, `TestCase`, `UiDriver`, `BY` selectors, `Step`, and device APIs.

## HDC and UiTest CLI

Official patterns:

```bash
hdc list targets
hdc -t <SN> shell
hdc shell aa start -a EntryAbility -b <bundleName> -m entry
hdc shell aa force-stop <bundleName>
hdc shell uitest screenCap -p /data/local/tmp/1.png
hdc shell uitest dumpLayout -p /data/local/tmp/1.json
hdc shell uitest uiInput click 100 100
hdc shell uitest uiInput doubleClick 100 100
hdc shell uitest uiInput longClick 100 100
hdc shell uitest uiInput swipe <from_x> <from_y> <to_x> <to_y> [velocity]
hdc shell uitest uiInput text "hello"
hdc shell uitest uiInput keyEvent BACK
```

`uitest` command-line capabilities include `help`, `screenCap`, `dumpLayout`, `uiRecord`, `uiInput`, `--version`, and `start-daemon`. `dumpLayout` supports saving to a path, including invisible controls, selecting attributes, filtering by bundle name or window ID, merging window information, and display selection on newer API versions.

First-time UI test execution on some devices may require enabling test mode:

```bash
hdc -n shell param set persist.ace.testmode.enabled 1
```

Do not run that automatically unless the user asks; it changes device test configuration.

## Hilog Debugging

Use hilog when screenshots and layout dumps show what happened but not why. It is useful after failed navigation, stale state, failed image/message send, native model errors, cloud route errors, file permission issues, or unexpected background task behavior.

Official hilog command patterns:

```bash
hdc shell hilog -h
hdc shell hilog -r
hdc shell hilog -z 300 -t app
hdc shell hilog -a 100 -t app
hdc shell hilog -z 500 -t app -L E
hdc shell hilog -z 500 -t app -P <pid>
hdc shell hilog -z 500 -t app -D <domain>
hdc shell hilog -z 500 -t app -e "MNN|Agent|Workflow|Exception|Error"
hdc shell hilog -w start
hdc shell hilog -w stop
hdc shell ls /data/log/hilog
hdc file recv /data/log/hilog <local_path>
```

Use the helper script for repeatable log capture:

```bash
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-clear
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-tail --lines 500 --type app --level D --regex "MNN|Agent|Workflow|Exception|Error" --out /tmp/harmony-hilog.txt
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-tail --lines 300 --type app --pid <pid> --out /tmp/harmony-app-hilog.txt
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py hilog-disk list
```

Keep these log rules explicit:

- Use `hilog-clear` immediately before a focused reproduction when possible, then run the UI action and `hilog-tail --out <file>` afterward.
- Prefer `--type app` and targeted `--level`, `--pid`, `--domain`, or `--regex` filters. Unfiltered hilog is noisy and expensive for an agent to analyze.
- Use `--out` for large captures, then summarize relevant lines instead of pasting complete logs into the final response.
- Do not receive `/data/log/hilog` full disk logs unless the user asks or approves; device-side logs can be large and may contain private data.

## App Sandbox File Transfer

Official HDC patterns for app sandbox transfer:

```bash
hdc file send -b <bundleName> <local-source> <remote-destination>
hdc file recv -b <bundleName> <remote-source> <local-destination>
hdc shell -b <bundleName> ls -A "./data/storage/el2/base"
hdc shell -b <bundleName> mkdir -p "./data/storage/el2/base/test"
```

Project defaults:

- Bundle: `com.example.mnnllmchat`
- Entry files root: `/data/storage/el2/base/haps/entry/files`
- Local sample root: `mobile_file_example/files`

Use the helper script instead of rewriting raw HDC commands:

```bash
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py snapshot --prefix /tmp/harmony-state --bundle com.example.mnnllmchat --merge-windows
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-ls /data/storage/el2/base/haps/entry/files/
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-mkdir /data/storage/el2/base/haps/entry/files/workflows
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-send mobile_file_example/files/workflows /data/storage/el2/base/haps/entry/files/
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-recv /data/storage/el2/base/haps/entry/files/workflows/ mobile_file_example/files/workflows/
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py file-recv /data/storage/el2/base/haps/entry/files/config.json ~/Desktop/config.json
```

Keep these transfer rules explicit:

- `file-send` can overwrite same-name files on the device. Ask before sending into app state directories such as `workflows`, `chat-sessions`, `memory`, or config files unless the user explicitly requested it.
- A directory source sent to `/data/storage/el2/base/haps/entry/files/` creates or merges that directory under `files/`, matching raw `hdc file send -b ... workflows .../files/` behavior.
- A remote directory ending in `/` pulls the directory contents into the local destination directory. Use this when mirroring `workflows/` without creating an extra nested folder.
- Pull single files to explicit local file paths, for example `~/Desktop/config.json`.
- `hdc shell -b` uses `./data/storage/...` paths in official examples. The helper normalizes `/data/storage/...` to `./data/storage/...` for `file-ls` and `file-mkdir`; raw `hdc file send/recv -b` commands can keep the absolute path style shown above.
- Pulling app data can expose private user data. Use narrow paths and summarize what was copied.

## Scriptization and Corrections Log

When a workflow repeats raw HDC/UiTest commands, add or reuse a helper subcommand in `scripts/harmony_hdc_ui.py`. Current reusable commands cover device diagnostics, app launch/stop, screenshot, layout dump, combined `snapshot`, hilog capture/filtering, UI input, and app sandbox file transfer.

Known corrections:

- `uitest dumpLayout -m` requires an explicit value on this device. Use `-m true`, implemented by `dump --merge-windows` and `snapshot --merge-windows`.
- `hdc shell -b <bundleName>` follows official `./data/storage/...` sandbox path examples. The helper normalizes `/data/storage/...` to `./data/storage/...` for `file-ls` and `file-mkdir`.
- `uv run` can fail inside the Codex sandbox when it cannot access `~/.cache/uv`. Request escalation for required UV commands instead of falling back to global Python.
- Raw sandboxed `hdc shell ...` may fail with connection issues even when the approved UV helper works. Prefer the helper for device operations.
- `dump --bundle <bundleName>` and `snapshot --bundle <bundleName>` can return an empty `[0,0][0,0]` tree when a system picker, camera, permission dialog, or other cross-bundle window is on top. Rerun without `--bundle` for full-window inspection.
- Hilog output is too noisy without filters. Prefer `hilog-tail --type app --level <D|I|W|E|F> --regex <pattern> --out <file>` or `--pid <pid>` when available.
- On the current device, `hilog -x` cannot be combined with `-z`, `-a`, `-t`, or other query filters; it prints `Mutlti commands can't be used in combination [CODE: -31]` while still returning success. The helper omits `-x` and uses `-z`/`-a` for finite captures.

Before finishing after a newly discovered command failure, update this section with the failed shape and the corrected helper behavior.

## ArkTS UiTest Pattern

Official UiTest examples use:

```ts
import { describe, expect, it, Level } from '@ohos/hypium';
import { abilityDelegatorRegistry, Driver, ON } from '@kit.TestKit';
import { UIAbility, Want } from '@kit.AbilityKit';
```

Core flow:

1. Create a fresh `Driver` in each test.
2. Resolve `bundleName` from `abilityDelegatorRegistry.getArguments().bundleName` unless a fixed bundle is required.
3. Start the app with `delegator.startAbility({ bundleName, abilityName: 'EntryAbility' })`.
4. Wait for UI stability with `driver.waitForIdle(...)` or `driver.delayMs(...)`.
5. Assert current top ability if launch correctness matters.
6. Find components with `ON.text(...)`, `ON.id(...)`, `ON.type(...)`, or other stable selectors.
7. `await` every UiTest API call.
8. After click/input/navigation, re-find components instead of reusing stale references.
9. Assert visible results with `driver.assertComponentExist(...)` and Hypium `expect`.
10. End with navigation cleanup such as `driver.pressBack()` when appropriate.

UiTest runs on real devices or emulators. Previewer is not a substitute for device UI automation.

## Hypium Python Pattern

Official Hypium Python examples use:

```python
from devicetest.core.test_case import TestCase, Step
from hypium import *

class Example(TestCase):
    def __init__(self, controllers):
        self.TAG = self.__class__.__name__
        TestCase.__init__(self, self.TAG, controllers)
        self.driver = UiDriver(self.device1)

    def setup(self):
        Step("回到桌面")
        self.driver.swipe_to_home()

    def process(self):
        Step("查找并点击控件")
        self.driver.touch(BY.text("信息"))

    def teardown(self):
        Step("停止应用")
        self.driver.stop_app("<bundleName>")
```

Use `BY.text(...)`, `BY.type(...)`, `find_component`, `find_all_components`, `find_window`, `touch`, `swipe`, `press_key`, and `stop_app` for device-side flows. Prefer semantic selectors over coordinates. Use coordinates only when layout dumps show no stable selector.

## Project UV Environment

This repository has a uv-created `.venv` with Python 3.14. Use it for all Python automation:

```bash
uv run --active python <script-or-module>
uv run --python .venv/bin/python python <script-or-module>
```

`uv run` executes commands in the project environment. `--active` tells uv to use the activated virtual environment when one is active; otherwise, use the repository `.venv` explicitly. If sandbox permissions block `~/.cache/uv`, request escalation instead of using global Python.

Install Hypium or host-side test dependencies into the project environment only after user approval if network/downloads are required.

## Missing Local Environment Suggestions

Diagnose before installing:

```bash
command -v uv
command -v hdc
uv run --active python --version
uv run --active python .agents/skills/harmony-hdc-ui-automation/scripts/harmony_hdc_ui.py doctor
hdc shell uitest --version
hdc shell hilog -h
```

Use these setup suggestions when a dependency is missing:

- **uv missing:** suggest installing uv on macOS with `brew install uv` when Homebrew is available. If Homebrew is unavailable, point the user to Astral's official uv installer and ask before running network installation commands.
- **Project `.venv` missing or broken:** suggest `uv venv --python 3.14 .venv`, then run project Python commands with `uv run --python .venv/bin/python python ...`.
- **hdc missing:** suggest installing DevEco Studio or Huawei command-line tools, then add the SDK toolchains directory to `PATH` or set `HDC=/path/to/hdc`. Verify with `hdc list targets`.
- **No connected device:** suggest starting a HarmonyOS emulator or connecting a real device with HDC debugging enabled, then verify with `hdc list targets`. When multiple devices are listed, require `--target <SN>`.
- **Device UiTest command unavailable:** suggest verifying HarmonyOS system/API support with `hdc shell uitest --version`. If official UiTest capabilities are not enabled on the device, ask before running persistent test-mode setup such as `hdc -n shell param set persist.ace.testmode.enabled 1`.
- **Hypium Python missing:** suggest obtaining the official DevEco Testing Hypium package that matches the user's DevEco/HarmonyOS version, then installing it into the project environment, for example `uv run --active python -m pip install /path/to/hypium-*.tar.gz`. Do not install into global Python.
- **ArkTS UiTest/Hvigor test environment missing:** suggest `ohpm install` and Hvigor sync with the repository's DevEco SDK/JBR environment. Use `$harmony-hvigor-unit-test` for the exact project commands and sandbox caveats.
- **Skill validation dependency missing:** run the validator with project Python first: `.venv/bin/python /Users/zhaoxi/.codex/skills/.system/skill-creator/scripts/quick_validate.py <skill-dir>`. If `yaml` is missing there, suggest `uv run --active python -m pip install PyYAML`.

Never auto-install packages, SDKs, or device test-mode changes during UI control work unless the user explicitly approves that setup step.
