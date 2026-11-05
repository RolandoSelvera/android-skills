---
name: aab-dex-checker
description: Audits Android App Bundles (.aab) for Android Vitals DEX code optimization signals. Measures uncompressed DEX size and inspects available R8 metadata or mapping information; use when reviewing an Android release bundle before Play Console submission.
metadata:
  author: Rolando Selvera
  keywords:
    - aab
    - dex
    - r8
    - obfuscation
    - google play console
    - android vitals
    - minification
    - bundle
---

## Overview

This skill provides a guided workflow to inspect an Android App Bundle (`.aab`) before publishing to Google Play Console. It measures the total uncompressed DEX code size and inspects available R8 metadata (`r8.json`) or mapping information to report Android Vitals optimization signals for shrinking, optimization, and obfuscation.

---

## Language

Use the language the user is already using. If it is unclear and materially affects the report, ask whether they prefer Spanish or English. Keep the report in that language.

---

## Step 1: Locate or Build the Release App Bundle (`.aab`)

1. Check if a release `.aab` exists in common locations:
   - `app/build/outputs/bundle/release/app-release.aab`
   - Root directory of the project
2. If the bundle is missing or the user requests a fresh build, ask how they want to proceed:
   - **Question:** "No recent app-release.aab was found. How would you like to proceed?" / "No se encontró un app-release.aab reciente. ¿Cómo deseas proceder?"
   - **Options:** `["Build automatically via Gradle", "I will build it manually in Android Studio"]`
3. If the user chooses an automatic build, use the project's Gradle wrapper and the correct module/task (for example, `./gradlew :app:bundleRelease`). Do not assume the module is named `app` when inspecting the project.

---

## Step 2: Measure Uncompressed DEX Size (Command 1)

Execute the shell command to calculate the total uncompressed size in bytes of all `.dex` files inside the bundle.

### Command for Git Bash / Linux / macOS
```bash
unzip -l app-release.aab | grep -E '\.dex$' | awk '{sum+=$1} END {print sum}'
```

### Command for Native Windows PowerShell
```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem; [System.IO.Compression.ZipFile]::OpenRead("app-release.aab").Entries | Where-Object { $_.FullName -like "*.dex" } | Measure-Object -Property Length -Sum | Select-Object -ExpandProperty Sum
```

### Interpretation
- Converts the byte count to MB (`Bytes / 1,000,000` or `Bytes / 1,048,576`).
- Android Vitals reports optimization metrics for all apps and games, but size-qualified alerts require at least **10 MB** uncompressed DEX for apps or **50 MB** for games. Report the applicable threshold and whether the bundle reaches it.
- Being below the size threshold means the bundle does not meet that alert's size criterion; it is not a general Google Play compliance pass. Reaching the threshold means optimization metrics may be relevant, not that the bundle fails.

---

## Step 3: Extract & Parse R8 Metadata (Command 2)

Inspect the R8 metadata JSON, when present, at `BUNDLE-METADATA/com.android.tools/r8.json`. It is included for supported builds using the latest patch of AGP 8.10 or higher; older AGP versions or other build systems may use `mapping.txt` or DEX heuristics for Android Vitals reporting. If metadata is absent, report that the local check is inconclusive and inspect available mapping data rather than treating absence as a failure.

### Command for Git Bash / Linux / macOS
```bash
unzip -p app-release.aab BUNDLE-METADATA/com.android.tools/r8.json
```

### Command for Native Windows PowerShell
```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem; $zip = [System.IO.Compression.ZipFile]::OpenRead("app-release.aab"); $entry = $zip.Entries | Where-Object { $_.FullName -eq "BUNDLE-METADATA/com.android.tools/r8.json" }; $reader = [System.IO.StreamReader]::new($entry.Open()); $reader.ReadToEnd(); $reader.Close(); $zip.Dispose()
```

### Metadata Interpretation
- Report the values actually present in `r8.json`, including the `options` and `stats` fields. Do not assume a fixed schema when fields are missing or differ between R8 versions.
- The Android Vitals bad-behavior thresholds are minimums of **25% each for optimization, obfuscation, and shrinking** for size-qualified bundles. A boolean such as `isObfuscationEnabled` only indicates whether the option is enabled; it is not by itself the measured percentage.
- `stats.noObfuscationPercentage` describes the portion reported as not obfuscated. Do not label `<25%` as Google's official cutoff: explain it as the unobfuscated share and, where the schema permits, relate it to the corresponding obfuscation percentage. Review broad keep rules when a large share remains unobfuscated.
- Do not treat `isDebugModeEnabled` as a Play compliance result. If reporting it, present the raw value as build metadata.

---

## Step 4: Deliverable Report

Present a clear, structured report to the user in their chosen language (English or Spanish).

### Report Structure

1. **Executive Summary / Resumen Ejecutivo:**
   - State whether the local checks meet the inspected signals, need attention, or are inconclusive. Do not claim this guarantees Google Play Console approval or a specific Android Vitals outcome.

2. **Metrics Breakdown / Desglose de Métricas:**
   | Metric / Métrica | Value / Valor | Target / Objetivo | Status / Estado |
   |---|---|---|---|
   | Total Uncompressed DEX Size | X.XX MB | 10 MB apps / 50 MB games for size-qualified alerts | Below / Meets threshold |
   | Shrinking, obfuscation, optimization | Report available metrics | Android Vitals minimum: 25% each when size-qualified | Meets / Needs attention / Unknown |
   | Unobfuscated Code (`noObfuscationPercentage`) | X.XX% | Interpret alongside the reported obfuscation metric | Informational |

3. **Detailed Findings & Recommendations / Hallazgos y Recomendaciones:**
   - Provide plain-language explanations of each metric and specific advice if any check failed or requires attention.

## References

- [Android Vitals: DEX code optimization](https://developer.android.com/google/play/vitals/code-optimization)
- [Android Vitals thresholds](https://developer.android.com/google/play/vitals)
