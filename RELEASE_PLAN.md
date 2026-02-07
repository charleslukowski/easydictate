# EasyDictate — Next Release Plan (v1.1.0)

## Current State (v1.0.0)

Working MVP with push-to-talk dictation, English-only Whisper base model, clipboard/auto-paste output, system tray integration, customizable hotkey, and a first-run wizard. No tests, no CI/CD, no changelog, no published GitHub releases.

---

## Planned Features

### 1. Multi-Language Support

**Why:** English-only is the single biggest limitation on who can use EasyDictate. Whisper supports 99 languages out of the box — we just need to expose it.

**What:**
- Replace the hardcoded `"en"` in `TranscriptionService.cs:58` with a configurable language setting
- Add a language dropdown to the Settings window (populated from Whisper's supported languages)
- Add an "Auto-detect" option that omits the language parameter and lets Whisper detect
- Download multilingual model variants (e.g., `ggml-base.bin` instead of `ggml-base.en.bin`) when a non-English language is selected
- Persist the language choice in `AppSettings`

**Scope:** `TranscriptionService.cs`, `SettingsService.cs`, `AppSettings.cs`, `SettingsWindow.xaml/.cs`, `ModelManager.cs`

---

### 2. Model Selection (Tiny / Base / Small / Medium)

**Why:** Users with powerful GPUs want better accuracy (larger models). Users on older hardware want faster responses (smaller models). Currently locked to `base.en`.

**What:**
- Add a model size selector to Settings: Tiny (~75MB), Base (~140MB), Small (~460MB), Medium (~1.5GB)
- Update `ModelManager` to download the selected model variant from Hugging Face
- Show estimated download size and transcription speed trade-offs in the UI
- Keep the current model cached — don't re-download if switching back
- Update the first-run wizard (step 4) to let users pick a model size

**Scope:** `ModelManager.cs`, `AppSettings.cs`, `SettingsWindow.xaml/.cs`, `FirstRunWindow.xaml/.cs`

---

### 3. Transcription History

**Why:** Transcriptions vanish after they hit the clipboard. Users expect to be able to review, search, and re-copy past dictations.

**What:**
- Add a local SQLite or JSON-file-based history store in `%APPDATA%\EasyDictate\history\`
- Save each transcription with timestamp, duration, text, and language
- Add a "History" window accessible from the system tray context menu
- Support search, copy, and delete operations
- Add a setting to disable history (privacy-conscious users)
- Auto-prune history older than a configurable retention period (default: 30 days)

**Scope:** New `HistoryService.cs`, new `HistoryWindow.xaml/.cs`, `AppSettings.cs`, `TrayIconViewModel.cs`

---

### 4. Audio Preprocessing — Voice Activity Detection (VAD)

**Why:** Users sometimes hold the hotkey a beat too long or release late, sending silence or noise to Whisper. VAD trims silence from the start and end, improving both speed and accuracy.

**What:**
- Add energy-based voice activity detection to `AudioCaptureService`
- Trim leading and trailing silence before sending audio to Whisper
- Optionally reject recordings that are entirely below the energy threshold (with "No speech detected" overlay feedback — already exists)
- Expose a sensitivity slider in Settings for noisy environments

**Scope:** `AudioCaptureService.cs`, `AppSettings.cs`, `SettingsWindow.xaml/.cs`

---

### 5. Update Checker

**Why:** No update mechanism exists. Users won't know about bug fixes or new features unless they manually check GitHub.

**What:**
- On startup (and optionally on a daily timer), check the GitHub Releases API for a newer version tag
- Compare against the current `Version` from the assembly
- Show a non-intrusive notification via the system tray balloon tip or overlay
- Link to the GitHub Releases page for download
- Add a "Check for Updates" option in the tray context menu
- Add a setting to disable automatic update checks

**Scope:** New `UpdateService.cs`, `App.xaml.cs`, `TrayIconViewModel.cs`, `AppSettings.cs`

---

### 6. File Logging & Diagnostics

**Why:** Current logging is `Debug.WriteLine` only — invisible to end users. When something goes wrong, there's no way to diagnose it.

**What:**
- Add file-based logging to `%APPDATA%\EasyDictate\logs\`
- Log application lifecycle, transcription events, errors, and performance metrics (transcription latency, audio duration)
- Rotate logs (keep last 7 days)
- Add a "Open Log Folder" option in the tray context menu
- Replace bare `catch { }` blocks with logged exceptions

**Scope:** New `LoggingService.cs` or integrate `Microsoft.Extensions.Logging`, touch most service files for log calls, `TrayIconViewModel.cs`

---

### 7. Improved Settings UI

**Why:** Some settings exist in the model (`MaxRecordingSeconds`, `PlaySounds`) but aren't exposed in the UI. Power users have to edit `config.json` manually.

**What:**
- Add a "Recording" section: max recording duration slider (10s–300s)
- Add a "Sound Effects" toggle (the setting exists, wire it up)
- Add a "Reset to Defaults" button
- Add tooltips/descriptions to every setting
- Group settings into tabs: General, Recording, Output, Appearance

**Scope:** `SettingsWindow.xaml/.cs`, `AppSettings.cs`

---

## Infrastructure & Quality

### 8. GitHub Actions CI/CD

- **Build workflow** (on every push): `dotnet restore` → `dotnet build` → `dotnet test`
- **Release workflow** (on tag `v*`): build → publish self-contained .exe → create GitHub Release with artifact attached
- Eliminates manual `publish.bat` + manual upload process

### 9. Unit Tests

- Add an `EasyDictate.Tests` xUnit project
- Priority test targets:
  - `SettingsService` — JSON round-trip, defaults, migration
  - `AppSettings` — default values, enum parsing
  - `ModelManager` — path construction, size validation
  - Hallucination filtering logic in `TranscriptionService`
- Target: 60%+ coverage on service logic (no hardware-dependent tests)

### 10. CHANGELOG.md

- Adopt [Keep a Changelog](https://keepachangelog.com/) format
- Document all changes from v1.0.0 through v1.1.0
- Automate changelog reminders in PR templates

---

## Release Checklist

1. Implement features (prioritized 1–7 above)
2. Write tests for new and existing code
3. Update version in `EasyDictate.csproj` to `1.1.0`
4. Write `CHANGELOG.md`
5. Manual smoke test on Windows 10 and Windows 11
6. Merge to `master`, tag `v1.1.0`
7. GitHub Actions builds and publishes the release automatically
8. Update website download link if needed

---

## Priority & Sequencing

| Priority | Feature | Effort | Impact |
|----------|---------|--------|--------|
| P0 | CI/CD + Tests (8, 9) | Medium | Foundation for everything else |
| P0 | CHANGELOG (10) | Small | Required for any release |
| P1 | Multi-Language (1) | Medium | Largest user base expansion |
| P1 | Model Selection (2) | Medium | Hardware flexibility |
| P2 | Transcription History (3) | Medium | Core UX improvement |
| P2 | Update Checker (5) | Small | User retention |
| P2 | File Logging (6) | Small | Supportability |
| P3 | VAD / Audio Preprocessing (4) | Medium | Quality improvement |
| P3 | Settings UI Improvements (7) | Small | Polish |
