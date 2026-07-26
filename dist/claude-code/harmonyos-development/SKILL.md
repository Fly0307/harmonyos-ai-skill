---
name: harmonyos-development
description: "Use when developing, reviewing, debugging, or migrating HarmonyOS NEXT apps with ArkTS, ArkUI, Stage model, DevEco Studio, module.json5, oh-package.json5, permissions, navigation, kits, testing, performance, or publishing."
---

# HarmonyOS Development

Use this skill for HarmonyOS NEXT native app work. Keep this file as the routing layer: read only the reference file(s) needed for the current task instead of loading the whole knowledge base.

Default working assumptions:
- Prefer Stage model, ArkTS, ArkUI, `UIAbility`, `module.json5`, `oh-package.json5`, HAP/HSP/HAR packaging, and DevEco Studio workflows.
- Treat `references/platform-versions.md` as a local snapshot. If the user asks for latest/current SDK, toolchain, release, or API behavior, verify against official Huawei documentation before relying on the snapshot.
- For production guidance, prefer the stable/release baseline documented in `references/platform-versions.md`; use preview/Beta material only when the user explicitly targets it.
- For project-specific work, inspect the repository first: `build-profile.json5`, `hvigorfile.ts`, `oh-package.json5`, `module.json5`, `entry/src/main/ets`, and relevant `.ets` files.
- For compiler errors, search the exact error text in `references/api-errors-gotchas.md` before proposing a fix.

## Official Source Lookup

Prefer official Huawei documentation when network or Context7 lookup is available.

| Source | Use |
|---|---|
| Huawei Developer Docs | `https://developer.huawei.com/consumer/cn/doc/` |
| HarmonyOS guide pages | Use page-specific Huawei docs under `developer.huawei.com/consumer/cn/doc/harmonyos-guides/...`; do not open the prefix as a standalone browser URL. For web search: `site:developer.huawei.com/consumer/cn/doc/harmonyos-guides <topic>`. |
| HarmonyOS API reference pages | Use page-specific Huawei docs under `developer.huawei.com/consumer/cn/doc/harmonyos-references/...`; do not open the prefix as a standalone browser URL. For web search: `site:developer.huawei.com/consumer/cn/doc/harmonyos-references <API or Kit>`. |
| Context7 Library IDs | `/websites/developer_huawei_consumer_cn_doc`, `/websites/developer_huawei_consumer_cn_doc_harmonyos-guides`, `/websites/developer_huawei_consumer_cn_doc_harmonyos-references` |

When using Context7, query the most specific Huawei library ID with the exact API, Kit, component, error text, and SDK/API version. When using web search, constrain searches to `developer.huawei.com/consumer/cn/doc`.

## How to Load Context

1. Classify the user request by topic.
2. Read the matching file(s) from `references/`.
3. If the topic is unclear, search references with `rg -n "keyword|API|symbol" references` and then read the smallest relevant file.
4. Do not read every reference file up front. Add more files only when the first file points to a cross-domain dependency.

## Reference Routing

| User request or topic | Read |
|---|---|
| HarmonyOS version, SDK/API baseline, DevEco version, API 23/24/26, HarmonyOS 6.1/6.1.1/7 | `references/platform-versions.md` |
| Project layout, Stage project setup, Hvigor, build profile, ohpm dependencies | `references/project-setup-build.md` |
| ArkTS syntax, strict checker, naming, performance rules, TaskPool, Worker, `@Concurrent`, `@Sendable` | `references/arkts-language.md` |
| ArkUI components, layout, animation, forms, dialogs, tabs, lists, responsive/foldable UI, keyboard, dark mode, gestures, immersive window | `references/arkui-ui.md` |
| Router, Navigation, NavPathStack, EventHub, decorators, StateStore, state management | `references/navigation-state.md` |
| UIAbility, Want, module declarations, permissions, startAbilityByType, background tasks, security, continuation, app links, shortcuts | `references/abilities-permissions.md` |
| Atomic services, meta-services, distributed capabilities, Form Kit service cards | `references/forms-cards-services.md` |
| Kit import names and official SDK category catalog | `references/kits-catalog.md` |
| relationalStore, preferences, resources, raw files, fileIo, document picker, cross-module resources | `references/data-storage-files.md` |
| HTTP, connectivity, WebSocket, ArkWeb, JS bridge, cookies, request.agent upload/download | `references/networking-web.md` |
| Photo picker, Camera Kit, Audio Kit, CoreSpeechKit, Scan Kit, AVSession, AVPlayer, Image Kit, Core Vision | `references/media-camera-audio-image.md` |
| Location Kit, Weather Service Kit, Map Kit | `references/maps-location-weather.md` |
| Notification Kit, Push Kit, Account Kit, Payment Kit, Share Kit | `references/push-payment-share-account.md` |
| arkxtest, JsUnit, UiTest, debugging tools, cold start, memory optimization, performance | `references/testing-debugging-performance.md` |
| HAP/HSP/HAR packaging, publishing, ArkGuard obfuscation | `references/packaging-publishing-obfuscation.md` |
| Known compile errors, SDK 6.0.1/API 21 fixes, common gotchas | `references/api-errors-gotchas.md` |
| Huawei docs links, GitCode sample projects, sample lookup by domain | `references/samples.md` |

## Common Cross-File Paths

- UI feature with navigation and state: read `references/arkui-ui.md` plus `references/navigation-state.md`.
- Permission-sensitive feature: read the feature-specific file plus `references/abilities-permissions.md`.
- Media or camera feature with background playback: read `references/media-camera-audio-image.md` plus `references/abilities-permissions.md`.
- Networked UI feature: read `references/networking-web.md` plus `references/arkui-ui.md`.
- Build or compile failure: read `references/project-setup-build.md`, `references/api-errors-gotchas.md`, then any file named by the failing API.

## Coordination With Device Automation

- Use `$harmony-hdc-ui-automation` for connected-device or emulator operations, screenshots, live
  UI trees, hilog capture, sandbox file transfer, exploratory input, and host-side Hypium Python.
- Load both skills when implementing a HarmonyOS change and validating it on a real device or
  emulator.
- Keep exact HDC and UiTest execution commands in the automation skill; keep architecture, ArkTS
  design, and SDK/API selection in this skill.

## Output Guidance

- Give ArkTS/ArkUI examples that match strict mode and the project SDK baseline.
- Prefer `Navigation` over legacy `Router` for new navigation code unless maintaining existing Router code.
- Include required `module.json5` permissions, `reason`, and `usedScene` when user-grant permissions are involved.
- Distinguish stable release guidance from preview/Beta guidance explicitly.
