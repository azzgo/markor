# Markor Agent Guide

This file is for agentic coding tools operating in this repository.
Follow the commands and conventions below to stay aligned with project
expectations and tooling.

--------------------------------------------------------------------------------
Build / Lint / Test
--------------------------------------------------------------------------------

Prereqs
- Android SDK installed and `ANDROID_SDK_ROOT` set.
- Use Android Studio for local dev when possible.
- Outputs, logs, and test results are placed in `dist/` by the Makefile.

Makefile shortcuts (preferred for day-to-day tasks)
- `make all`                     # spellcheck + lint + deptree + test + build + aapt
- `make lint`                    # Android lint for DefaultDebug flavor
- `make test`                    # Unit tests for DefaultDebug flavor
- `make build`                   # Assemble APKs (default flavor = Atest)
- `make install`                 # Install APKs from `dist/` on device
- `make run`                     # Run installed app on device
- `make clean`                   # Gradle clean + remove build/dist artifacts

Flavor selection
- Makefile uses `FLAVOR` and defaults to `Atest`.
- Example: `FLAVOR=Default make build` (note: flavor names are `flavorAtest`,
  `flavorDefault`, `flavorGplay` in Gradle, Makefile expects just `Atest`/`Default`/`Gplay`).

Gradle equivalents (run from repo root)
- Build APK: `./gradlew :app:assembleFlavorAtest -x lint`
- Lint: `./gradlew :app:lintFlavorDefaultDebug`
- Unit tests: `./gradlew :app:testFlavorDefaultDebugUnitTest -x lint`

Run a single test (Gradle)
- Class: `./gradlew :app:testFlavorDefaultDebugUnitTest --tests "com.example.FooTest"`
- Method: `./gradlew :app:testFlavorDefaultDebugUnitTest --tests "com.example.FooTest#testName"`

Notes
- Gradle is configured to emit HTML+XML test reports.
- The Makefile writes Gradle logs to `dist/log/` and validates build success.

--------------------------------------------------------------------------------
Code Style and Conventions
--------------------------------------------------------------------------------

General
- Primary language is Java. Kotlin is disabled by default (`enable_plugin_kotlin = false`).
- Follow AOSP Java Code Style: https://source.android.com/source/code-style
- Use Android Studio "Reformat" and "Optimize Imports" before submitting.
- Target is Java 8 (see `compileOptions` in `app/build.gradle`).
- Encoding is UTF-8 for source files.

Imports
- Keep import order consistent with existing files:
  Android / AndroidX first, then project packages, then `java.*` / `javax.*`, then others.
- Remove unused imports; keep explicit imports (no wildcard imports).

Formatting
- 4 spaces per indent; no tabs.
- Braces on same line; one true brace style.
- Keep lines reasonably short; wrap method calls and conditionals for clarity.
- Add blank lines between logical blocks as seen in existing classes.

Naming
- Classes / interfaces: `PascalCase`.
- Methods / variables: `camelCase`.
- Constants: `UPPER_SNAKE_CASE`.
- Private instance fields often use a leading underscore (e.g., `_toolbar`).
  Keep this convention when modifying existing classes.
- Resource IDs use Android conventions (snake_case), and are referenced via `R`.

Types and null handling
- Prefer `final` for parameters and locals when not reassigned.
- Use guard clauses / early returns for null checks and invalid inputs.
- When using boxed types (`Integer`, `Boolean`), document nullable behavior with
  local variable names and checks before use.

Error handling
- Favor early validation and user-facing messages over silent failures.
- Use try/catch only where recovery is meaningful; avoid swallowing exceptions.
- Log with `Log` when needed for diagnostics, but keep logs concise.

Android specifics
- Use AndroidX libraries (project already depends on AndroidX).
- Keep UI changes consistent with existing XML resources and styles.
- Prefer string resources over hardcoded UI strings.

Testing
- Unit tests use JUnit4 and AssertJ (`testImplementation` in `app/build.gradle`).
- Keep tests deterministic and avoid relying on device state.

Project structure notes
- Main app module: `app/`.
- Java source root: `app/src/main/java/`.
- Assets/templates: `app/src/main/assets/`.
- Raw resources copied from repo root into `app/src/main/res/raw` via Gradle task.

--------------------------------------------------------------------------------
No Cursor / Copilot rules found
--------------------------------------------------------------------------------

- `.cursor/rules/` not present.
- `.cursorrules` not present.
- `.github/copilot-instructions.md` not present.

--------------------------------------------------------------------------------
When in doubt
--------------------------------------------------------------------------------

- Follow existing file patterns and naming in the same package.
- If changing formatting or imports, use Android Studio auto-format to match
  the established style.
- Prefer Makefile targets unless you need a specific Gradle task.
