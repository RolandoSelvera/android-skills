---
name: database-export
description: Exports the local database (Room, raw SQLite, or Realm) of an Android app to the developer's computer, from a device or emulator, without modifying the app's business logic. Use whenever the user asks to export, dump, pull, extract, back up, or inspect a Room database, a Realm database, a .db/.sqlite file, or app data off a device — for debug, release, or a specific product flavor. It asks which build variant, which device/emulator, and where and under what name to save the export, auto-detects the database engine and file names from Gradle and source code, pulls files over adb via run-as with zero code changes, launches the app only if it never produced the database yet, and for non-debuggable release builds temporarily rebuilds a debuggable variant to extract data then reverts project and device to their original state. Prefer this over improvising ad-hoc adb or root commands whenever the user wants to inspect stored app data, compare data across environments, or hand a database to QA or support.
metadata:
  author: Rolando Selvera
  keywords: [room, sqlite, realm, adb, run-as, database export, debuggable, build variant, product flavor, device selection, android studio, windows powershell binary corruption]
---

# Database Export

## Overview

This skill pulls an Android app's on-device database to the developer's machine: Room, a hand-rolled `SQLiteOpenHelper`, or Realm (Java/legacy or Kotlin SDK). The guiding idea is to avoid touching the app's code whenever possible — `run-as` already gives read access to a debuggable app's private storage, so most exports need no rebuild and no extra run of the app. The one case that genuinely requires rebuilding something is a non-debuggable release build, where this skill temporarily flips `isDebuggable` on, extracts, and puts the project and the installed app back exactly as they were.

Do all of this interactively: confirm the variant, the device, the detected database(s), and the save location and name with the user before copying anything, since guessing wrong on a release build means rebuilding twice for nothing, and guessing wrong on a file name means a confusing overwrite later.

## Asking the User

Four points below need a real answer from the user before the skill continues: **which build variant**, **which device or emulator**, **where to save the export**, and **what to name it**. None of these should ever be answered with a silently-picked default — if the user hasn't said, ask; if they already said earlier in the conversation, surface that as the suggested/default candidate and still get it confirmed rather than skipping the question.

If the agent running this skill exposes a structured question/elicitation tool — one that presents the user with selectable options instead of a plain-text question, such as Claude Code's `AskUserQuestion` or an equivalent tool in another agent — use it for all four, passing the candidates listed in each step as the options, plus a free-text/"other" choice wherever a step proposes names. If no such tool is exposed, ask the same question in plain chat text, spelling out the same candidates.

## Step 1: Ask Which Build Variant to Target

1. Read `app/build.gradle` / `app/build.gradle.kts` for `buildTypes { }`, `productFlavors { }`, and `flavorDimensions`. Note, for each `buildType`, whether `debuggable`/`isDebuggable` is `true` or left at its default (`true` for `debug`, `false` for `release` unless overridden), and any `applicationIdSuffix`.
2. Build the list of variants that actually exist (every flavor × every build type, e.g. `devDebug`, `stagingRelease`, `prodRelease`).
3. Ask which environment to export from, with the detected variants as candidates (see "Asking the User" above) — don't type a build type blind, since a project with no flavors just has `debug`/`release`.
4. Resolve the final `applicationId` for that variant (base `applicationId` + any `applicationIdSuffix` from the flavor and/or build type) — this is what `run-as` and `pm` need.

Keep the user's answer around; every later step is scoped to this one variant.

## Step 2: Detect the Database Engine and File Names

Grep the module's source and Gradle files rather than assuming one engine — a project can use more than one.

- **Room**: dependency on `androidx.room:room-runtime` / `room-ktx`. Find classes annotated `@Database` and read the string literal passed to `Room.databaseBuilder(context, XxxDatabase::class.java, "name.db")` (or `.java`) for the real file name. Room stores it under `databases/<name>.db`, usually in WAL mode, so also look for `<name>.db-wal` and `<name>.db-shm` next to it.
- **Raw SQLite**: dependency-free subclasses of `SQLiteOpenHelper`; the file name is the constructor's `DATABASE_NAME` argument (or the literal passed to `super(context, "name.db", ...)`). Same `databases/` location and WAL sidecar files.
- **Realm**: dependency on `io.realm:realm-android-library` (legacy Java SDK) or `io.realm.kotlin:library-base` (Realm Kotlin SDK). Find `RealmConfiguration.Builder().name("x.realm")`; if no `.name(...)` call appears, the default is `default.realm`. Realm stores its file under `files/`, not `databases/`, and keeps a `.management/` folder and a `.lock` file alongside it that don't need to be copied.

End this step with a concrete list: `{ engine, file name, on-device path relative to the app's data directory }` for every database the app has. Show it to the user and let them narrow it down if they only want one.

## Step 3: Ask Which Device or Emulator to Use

```bash
adb devices -l
```

1. For every attached device/emulator, resolve a human-readable label: `adb -s <serial> shell getprop ro.product.model` for a physical device, or the AVD name already shown in `devices -l` for an emulator.
2. Ask the user which one to use, with every attached device/emulator as a candidate — **even when only one is attached**, since silently picking "the only one" is still picking without asking. If the user already named a specific device or emulator earlier in the conversation, list it as the suggested/default candidate so confirming it is one click, but still ask.
3. From here on, every `adb` command in this skill targets that device explicitly with `-s <serial>`.

If no device is attached, or the chosen one doesn't show `device` as its state in `adb devices`, stop and tell the user before going any further.

## Step 4: Confirm the Package Is Installed

```bash
adb -s <serial> shell pm list packages <applicationId>
```

If it isn't installed yet, install the APK for the chosen variant first (building it is covered by the project's normal build task, not this skill) — this skill exports an already-installed app's data, it doesn't create the app.

## A Note on Windows: Binary-Safe Redirection

Every copy in Steps 5 and 6 redirects `adb exec-out`'s output straight to a file with `>`. That only stays byte-for-byte identical to the on-device file if the shell doing the redirection passes bytes straight through without reinterpreting them as text. `cmd.exe` does this correctly. **PowerShell's default `>` (an alias for `Out-File`) does not** — it captures the command's output as a .NET string, re-encodes it, and silently corrupts binary files in the process (nulls get dropped, high bytes get mangled, line endings get rewritten). A Realm file corrupted this way typically fails to open with an error like `file has an invalid size (...)`; a Room/SQLite file corrupted the same way can be worse, since it sometimes still opens but with subtly wrong data.

If this agent is running on Windows and its shell is PowerShell (`powershell.exe` or `pwsh`), wrap every `exec-out` copy in Steps 5 and 6 through `cmd.exe` instead, e.g.:
```
cmd /c "adb -s <serial> exec-out run-as <applicationId> cat files/default.realm > C:\path\to\default.realm"
```
If the agent's shell is already `cmd.exe`, bash, zsh, or a POSIX shell inside WSL/Git Bash, this isn't needed — only native PowerShell redirection has this problem.

**Verify every copy regardless of platform or shell**, right after making it: compare the device-side byte count to the local file's byte count before trusting it.
```bash
adb -s <serial> shell run-as <applicationId> wc -c databases/app.db   # or files/default.realm for Realm
```
If the two sizes don't match, the copy is corrupt — don't hand it to the user or move on to Step 7. Re-copy it (using the `cmd.exe` wrapper above if the first attempt used PowerShell's `>`) and check again.

## Step 5: Export From a Debuggable Variant (the common case)

This covers `debug` and any custom build type left with `debuggable true` — which is most variants most of the time.

1. **Check whether the database file already exists** (meaning the app already ran at least once and nothing further needs to run now):
   ```bash
   adb -s <serial> shell run-as <applicationId> ls databases/ 2>/dev/null
   adb -s <serial> shell run-as <applicationId> ls files/ 2>/dev/null
   ```
2. **Only if the expected file is missing**, launch the app once so it creates its database, give it a few seconds, then stop it so buffers flush to disk:
   ```bash
   adb -s <serial> shell monkey -p <applicationId> -c android.intent.category.LAUNCHER 1
   sleep 5
   adb -s <serial> shell am force-stop <applicationId>
   ```
   Skip this entirely when the file is already there — that's the whole point of checking first.
3. **If the app is currently running in the foreground**, stop it before copying so WAL/journal data is flushed to the main file instead of sitting in a sidecar:
   ```bash
   adb -s <serial> shell am force-stop <applicationId>
   ```
   Mention to the user that the app will restart normally the next time they open it; nothing is lost.
4. **Copy every file you found in Step 2** with `exec-out` (not `adb pull`, which can't reach another app's private storage) into a local temp directory, using the binary-safe redirection from the note above:
   ```bash
   adb -s <serial> exec-out run-as <applicationId> cat databases/app.db     > /tmp/db-export/app.db
   adb -s <serial> exec-out run-as <applicationId> cat databases/app.db-wal > /tmp/db-export/app.db-wal
   adb -s <serial> exec-out run-as <applicationId> cat databases/app.db-shm > /tmp/db-export/app.db-shm
   # Realm:
   adb -s <serial> exec-out run-as <applicationId> cat files/default.realm  > /tmp/db-export/default.realm
   ```
   Missing `-wal`/`-shm` files are normal (they only exist while there's uncheckpointed data) — don't treat a failed copy of those two as an error. For every file that *should* exist, run the byte-count check from the note above before trusting the copy.

## Step 6: Export From a Non-Debuggable Release Variant

`run-as` only works against a debuggable app, and a standard `release` build type isn't one. Rather than requiring root, temporarily make this one build debuggable, reinstall it over itself (same signing certificate, so the install upgrades in place and keeps the existing user data), extract, then put everything back.

1. **Back up the build file** before touching it: `cp app/build.gradle.kts app/build.gradle.kts.bak` (or `.gradle` for Groovy).
2. **Add one line** to the variant's `buildTypes` block — don't touch `signingConfig`, `minifyEnabled`, or anything else, since changing the signing certificate would break the in-place reinstall this whole approach depends on:
   - Kotlin DSL: `isDebuggable = true`
   - Groovy: `debuggable true`
3. **Rebuild only that variant**, e.g. `./gradlew assembleRelease` or `./gradlew assemble<Flavor>Release`.
4. **Reinstall over the existing app, keeping its data**:
   ```bash
   adb -s <serial> install -r -d app/build/outputs/apk/release/app-release.apk
   ```
   This requires the same signing keystore the app is currently installed with. If that keystore isn't available locally (common when release signing happens only in CI), stop here and tell the user: the only remaining option is a rooted device/emulator, which this skill does not attempt on its own.
5. **Extract** using the exact commands and the byte-count verification from Step 5 (and the Windows redirection note above it).
6. **Revert the project**: restore the backup (`mv app/build.gradle.kts.bak app/build.gradle.kts`), rebuild the original release APK, and reinstall it the same way (`-r -d`) so the device ends up back on the real, non-debuggable production build. Don't skip this — leaving a production-signed build debuggable on a device is a real security regression, not just a loose end.

## Step 7: Ask Where to Save and What to Name the File(s)

This step always involves two real questions — never skip either one, even when the destination seems obvious from earlier in the conversation:

1. **Destination folder.** If the user already gave one earlier (e.g. "guárdala en ~/Desktop/notas-debug/"), surface it as the suggested candidate and get it confirmed; otherwise propose a default like `./exported-databases/<variant>/` as one of the candidates and ask.
2. **File name.** Propose a few candidates — the database's original on-device name, a variant that bakes in the build variant and today's date (e.g. `app_stagingDebug_2026-11-07.db`), and a free-text/"other" option — and ask which to use. When exporting a database plus its `-wal`/`-shm` sidecars, or more than one database at once, ask once for a base name and keep each file's own extension/suffix.

Use the agent's structured ask tool for both questions if one is available (see "Asking the User" above); otherwise ask in plain text. Only once both are confirmed, copy the files from the temp directory to the final destination with the chosen name(s), and clean up the temp directory.

## Step 8: Verify the Export

- **Room / raw SQLite**, if `sqlite3` is available on the developer's machine (not on the device — recent Android images don't ship it):
  ```bash
  sqlite3 app.db "PRAGMA wal_checkpoint(FULL);"   # folds any copied -wal file into app.db
  sqlite3 app.db ".tables"
  ```
- **Realm**: there's no `sqlite3`-equivalent CLI check, so the byte-count comparison from the Windows redirection note (just before Step 5) is the only automated integrity check available — if that passed, tell the user to open the `.realm` file in Realm Studio as a final confirmation that it reads correctly.
- Report the final file paths and sizes back to the user, and, if Step 6 was used, explicitly confirm the device is back on the original release build.
