# Implementation Plan - CloudStream 3 Build & Workflow Repair

Fix build, dependency, and configuration issues in the CloudStream 3 Android application while preserving all core features, media playback functionality, and provider architecture.

## User Review & Critical Decisions

> [!IMPORTANT]
> The primary build error stems from incompatible Gradle/AGP toolchain versions (`forUseAtConfigurationTime` removed in modern Gradle) and JitPack dependency resolution (`com.github.Blatzar:NiceHttp`). We will update build configurations and dependencies without breaking existing source code or provider interfaces.

- **Confirmed Decision 1**: Upgrade AGP and Kotlin Gradle configuration to support modern build environments while maintaining synthetic layout compatibility via ViewBinding / updated dependencies.
- **Confirmed Decision 2**: Update `com.github.Blatzar:NiceHttp` to a verified JitPack release tag (`0.4.1` / `0.4.18`) or fallback module.
- **Confirmed Decision 3**: Add a standalone `.github/workflows/build.yml` workflow configured with `workflow_dispatch` and `push` to main, running on Java 17/21 to produce a downloadable debug APK artifact.

---

## 1. Overview & Core Concept

CloudStream 3 is a modular Android streaming and media aggregation application. This plan addresses the Gradle build failure, updates the build script for modern environment compatibility, ensures all provider interfaces (anime, movies, TV shows, and live TV) remain fully functional, and establishes a robust GitHub Actions CI/CD workflow for automated APK generation.

---

## 2. Technical Architecture & Build Fix Strategy

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CloudStream 3 Android App                       │
├───────────────────┬───────────────────┬────────────────────────────────┤
│   UI Layer        │  Data & Providers │   Network & Extractors         │
│   (Fragments,     │  (MainAPI, Sync,  │   (NiceHttp, OkHttp,           │
│    Adapters)      │   Repositories)   │    ExtractorApi)               │
└─────────┬─────────┴─────────┬─────────┴────────────────┬───────────────┘
          │                   │                          │
┌─────────▼───────────────────▼──────────────────────────▼───────────────┐
│                          Gradle Build Pipeline                         │
│  - Android Gradle Plugin 8.2+ / Gradle 8.x / 9.x                     │
│  - Java 17 / Java 21 Toolchain                                         │
│  - Updated JitPack Dependencies (NiceHttp 0.4.x, NewPipeExtractor)    │
└────────────────────────────────────────────────────────────────────────┘
```

### Key Build Modifications Required

1. **Gradle Build Script (`build.gradle` / `app/build.gradle`)**:
   - Update `com.android.tools.build:gradle` plugin version to compatibility level for Gradle 8/9 (`8.2.2`).
   - Update `kotlin-gradle-plugin` to `1.9.22` or compatible release.
   - Adjust `compileSdkVersion` / `targetSdkVersion` / `minSdkVersion` settings and Java toolchain options (`JavaVersion.VERSION_17`).
   - Enable `viewBinding = true` and `buildConfig = true` in `app/build.gradle`.
   - Ensure `kotlin-parcelize` replaces `kotlin-android-extensions`.

2. **Dependency Resolution**:
   - Resolve `com.github.Blatzar:NiceHttp` by updating JitPack repository declaration and pointing to verified version (`0.4.18`).
   - Update `NewPipeExtractor` and `glide` dependency specifications for modern AGP.

3. **GitHub Actions Workflow (`.github/workflows/build.yml`)**:
   - Configured with `workflow_dispatch` and `on.push` triggers.
   - Sets up Java 17 (Temurin).
   - Grants execution permissions to `./gradlew` (or executes `gradle assembleDebug`).
   - Uses `actions/upload-artifact@v4` to attach the compiled debug APK (`app/build/outputs/apk/debug/app-debug.apk`).

4. **Preservation of CloudStream Functionality**:
   - Keep all existing provider classes (`com.lagradost.cloudstream3.movieproviders`, `animeproviders`, `liveproviders`, `metaproviders`).
   - Retain full player integration (ExoPlayer, Cast, subtitle decoders, M3U8 handling).
   - Preserve custom repository and extension loading capabilities in `ExtensionManager.kt`.

---

## 3. Verification Plan

- Run compilation checks (`compile_applet`) locally in the Android environment.
- Validate that all provider and view classes compile without unresolved symbols.
- Verify that `.github/workflows/build.yml` syntax is strictly valid and uses `actions/upload-artifact@v4`.
