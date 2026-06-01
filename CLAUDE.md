# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lawnchair is a free, open-source Android home app (launcher) based on AOSP Launcher3 from Android 16. It adds Pixel Launcher features and rich customization on top of the stock launcher. This branch (`16-dev`) is Lawnchair 16, currently in development.

## Build Commands

**Prerequisites:** JDK 21, Android SDK with compile SDK 37.

```bash
# Build debug APK (recommended development variant)
./gradlew assembleLawnWithQuickstepGithubDebug

# Build release APK
./gradlew assembleLawnWithQuickstepGithubRelease

# Build nightly variant
./gradlew assembleLawnWithQuickstepNightlyRelease

# Build Play Store variant
./gradlew assembleLawnWithQuickstepPlayDebug
```

## Code Quality

```bash
# Run Spotless linter (required to pass CI)
./gradlew spotlessCheck

# Auto-fix formatting issues
./gradlew spotlessApply

# Lint check
./gradlew lint
```

Spotless enforces:
- **Java:** Google Java Format (AOSP style)
- **Kotlin:** ktlint with Compose rules (only on `lawnchair/src/**/*.kt`)

## Testing

```bash
# Run all unit tests
./gradlew test

# Run tests for a specific variant
./gradlew testLawnWithQuickstepGithubDebugUnitTest

# Run a single test class
./gradlew testLawnWithQuickstepGithubDebugUnitTest --tests "com.android.launcher3.example.TestClass"
```

Test sources are in `tests/src/`, `tests/shared/`, and `tests/multivalentTests/`. Test frameworks: JUnit 4, Mockito, Google Truth, Robolectric.

## Module Structure

The root project is the Android application module (`com.android.application`). Lawnchair code lives in `lawnchair/src/` as a source set, not a separate Gradle module.

**Key source sets:**
- `src/` — Modified AOSP Launcher3 (Java/Kotlin)
- `lawnchair/src/` — Lawnchair customization layer (Kotlin)
- `quickstep/src/` — Recents/QuickStep gesture integration
- `compose/` — Jetpack Compose facade and features

**Library modules:**
- `:iconloaderlib`, `:searchuilib`, `:animationlib`, etc. — AOSP-forked from `platform_frameworks_libs_systemui/` (git submodule)
- `:shared`, `:plugin`, `:animation`, `:unfold`, etc. — SystemUI forks from `systemUI/`
- `:compatLib` + per-Android-version submodules — QuickSwitch compatibility (root required)
- `:dagger` — Shared DI annotations
- `:concurrent` — Executor/DI bindings (Hilt)
- `:wmshell` — Window Manager Shell
- `:flags` — Feature flags
- `:modules:widgetpicker` — Widget picker (only module using Kotlin DSL `build.gradle.kts`)

## Build Variants

Three flavor dimensions:

| Dimension | Flavors | Purpose |
|-----------|---------|---------|
| `app` | `lawn` | Lawnchair customization |
| `recents` | `withQuickstep` | Gesture navigation (minSdk 26) |
| `channel` | `github`, `nightly`, `play` | Distribution channel |

**Application IDs:** `app.lawnchair` (github), `app.lawnchair.nightly` (nightly), `app.lawnchair.play` (play)

The default development variant is `lawnWithQuickstepGithubDebug`.

## Architecture

**Inheritance chain:** `LawnchairLauncher` → `QuickstepLauncher` → `Launcher` → `BaseActivity`

**Dependency Injection:** Layered Dagger architecture. `:dagger` module provides scope annotations (`@ApplicationContext`, `@LauncherAppSingleton`, `@ActivityContextSingleton`). `LauncherBaseAppComponent` is the base DI component; `QuickstepBaseAppComponent` extends it with recents dependencies. `:concurrent` and `:wmshell` use Hilt.

**Preferences (dual-layer):**
- Legacy: `PreferenceManager` — SharedPreferences-based (`lawnchair/src/app/lawnchair/preferences/`)
- Modern: `PreferenceManager2` — DataStore-based with Flow/StateFlow (`lawnchair/src/app/lawnchair/preferences2/`)

**Data layer:** Room database in `lawnchair/src/app/lawnchair/data/` with Entity/DAO/Service pattern. Schemas stored in `schemas/`.

**UI:** Jetpack Compose for preferences screens (`lawnchair/src/app/lawnchair/ui/preferences/`), Material 3.

**Where to add new code:**
- New Lawnchair features → `lawnchair/src/app/lawnchair/`
- Launcher3 modifications → `src/` (keep to a minimum)
- New Compose features → `compose/features/`

## Commit Convention

We follow **Conventional Commits**: `type(scope): subject`

Allowed types: `feat`, `fix`, `style`, `refactor`, `perf`, `docs`, `test`, `chore`

Example: `feat(settings): Add toggle for new feature`

All PRs target the `16-dev` branch.

## Key Configuration

- **Version catalog:** `gradle/libs.versions.toml`
- **Gradle properties:** `gradle.properties` (JVM `-Xmx4g`, parallel builds, config cache enabled)
- **Compose stability config:** `compose_compiler_config.conf`
- **ProGuard rules:** `proguard.pro` + per-module `proguard.flags`
- **Submodule:** `platform_frameworks_libs_systemui/` (run `git submodule update --init --recursive` if library modules show errors)
