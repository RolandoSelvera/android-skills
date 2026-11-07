---
name: android-video-automation
description: Explore an Android app, interact with a physical device or emulator, and capture a requested UI flow as a video. Use when recording or reproducing Android app workflows.
metadata:
  author: Rolando Selvera
  short-description: Explore and record Android UI flows
  keywords:
    - Android
    - ADB
    - emulator
    - screen recording
    - UI automation
---

# Android Video Automation

Explore and run the user's requested Android app flow, capture it as a video, and save the verified recording to the location they choose. Do not require Android Studio. Adapt to the tools and host operating system available to the current agent.

## Guided start

1. Inspect the project context quietly before asking questions. Look for applicable `AGENTS.md` files, README or project instructions, app modules, package/application IDs, launch configuration, and existing test or automation conventions. Do not assume a particular build system or module name.
2. Make the first question to the user: where to save the video. Use the agent's structured ask/input tool when available; otherwise ask in the conversation. Do not choose a destination on the user's behalf.
3. Discover the available physical devices, running emulators, and existing AVDs, then ask which exact target the user wants to use. Show useful identifiers such as model, serial, AVD name, and whether it is running. Never silently select a connected device, even when it is the only one. If the user already named a target, confirm it against discovery rather than asking again. Ask before launching a stopped AVD; do not create an AVD or install tooling without authorization.
4. Inspect the project's actual build variants or equivalent run configurations, then ask which one to execute. Present discovered choices (for example, `debug`, `release`, or a project flavor such as `stage`) and do not infer a choice from build defaults. If variants cannot be discovered, ask the user what to run.
5. Ask what filename to use for the video, preferably with the agent's structured ask/input tool; if unavailable, ask in the conversation. Reuse a filename the user already specified.
6. Resolve any ambiguity about the flow before interacting or recording. Ask targeted questions when the starting activity, screen, fragment, destination, or a choice between paths (such as X or Y) is unclear. Do not guess; wait for the user's answer before proceeding.
7. Ask only for other details that cannot be established from the project or request, such as desired pacing or non-sensitive data needed for the flow. Never request passwords or secrets; let the user enter them directly on the device when needed.
8. Use the user's language for questions and the completion summary.

## Discover tools and select a target

- Check which native Android/device automation tools the agent provides. Use them when available, following their documented interaction and recording capabilities.
- Inspect the host for usable Android tooling, including `adb`, Android CLI (`android`), `emulator`, and `avdmanager`, plus SDK environment variables and common SDK locations for the host OS. Use the shell syntax appropriate to the detected operating system; do not assume PowerShell or a particular IDE.
- If ADB is available, check its version and list attached devices. If emulator tooling is available, list existing AVDs. Diagnose command failures and report the actual limitation instead of treating an installed executable as a working device setup.
- Present all usable connected devices and existing AVDs as choices; require the user's selection before launching the app. If no target is available, explain the options and ask whether to start an existing AVD or use the script fallback. Do not assume the attached device is the intended target.
- Determine whether the requested app is already installed and launchable on the selected target. If building or installing the app is necessary, use the user-selected variant, inspect project instructions, and ask before making a material project/device change not already requested.

## Explore and run the flow

- Verify the user-confirmed starting screen and each important transition using the best available live UI inspection, accessibility hierarchy, screenshot, or device automation tool. Prefer semantic selectors when supported. Use coordinates only when needed and ground them in a current screenshot or inspected element bounds.
- When only ADB is available, use the device's supported inspection and input capabilities (for example, UI hierarchy dumps, screenshots, and `adb shell input`). Never invent selectors, coordinates, wait times, or app behavior. Ask for clarification if an unverified step is essential.
- Use explicit waits tied to observed transitions where possible. Keep the requested flow faithful; do not add unrelated navigation or change app data beyond what the user requested.

## Capture and deliver

- Use the output directory and filename chosen during the guided start before beginning any recording. Capture with the agent's native recorder when available; otherwise use ADB `screenrecord` if the selected target supports it. Handle host paths for the detected operating system.
- Stop recording reliably after the flow, including on interaction or transfer failure. Pull or export the video to the requested location, verify that the output exists and is non-empty, and remove a temporary device-side copy only after verification.
- If direct recording cannot run because a tool or device is missing, prepare a runnable script or commands tailored to the tools that are actually present. Mark unresolved values and prerequisites clearly; never claim a video was captured unless the output file was verified.
- Finish with the verified interaction path, output path, and any remaining limitation. Keep the summary concise.
