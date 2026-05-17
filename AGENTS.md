# Build Commands

```bash
./gradlew assembleDebug    # debug APK → app/build/outputs/apk/debug/app-debug.apk
./gradlew assembleRelease  # signed release APK → app/build/outputs/apk/release/app-release.apk
```

There are no lint or test commands worth running — the project has no unit tests and no custom lint config.

# Configuration

All app-specific values live in **`app.properties`** (project root):

```properties
app.url=https://example.com/     # URL loaded on launch → BuildConfig.HOME_URL
app.package=com.example.myapp   # applicationId
app.name=My App                  # resValue string/app_name
```

`app/build.gradle.kts` reads these at build time and injects them as `BuildConfig` fields and `resValue` strings. **Nothing in the Kotlin source needs editing** when customizing the app.

For release signing, create **`keystore.properties`** (gitignored) with `storeFile`, `storePassword`, `keyAlias`, `keyPassword`. Place the `.p12` keystore at `keystore/release-signing.p12` (or adjust `storeFile`). If `keystore.properties` is absent, release builds are unsigned.

# Architecture

The entire app is a single activity: `app/src/main/java/io/github/webviewtemplate/MainActivity.kt`.

Key behaviors implemented in `MainActivity`:

- **Edge-to-edge layout** with `WindowCompat.setDecorFitsSystemWindows(window, false)` and manual inset padding applied to the root `FrameLayout`.
- **Status bar color sync**: after each page load, JS evaluates `document.body`/`documentElement` background color; `MainActivity.syncTopInsetColorWithPage()` parses the CSS color and sets the root container background + light/dark status bar icons via `WindowInsetsControllerCompat`.
- **Back navigation**: intercepts back press to call `webView.goBack()` before finishing the activity.
- **File upload**: `onShowFileChooser` builds a chooser `Intent` combining the web `<input accept>` types, a gallery picker, and a camera capture intent (using `FileProvider` with authority `${packageName}.fileprovider`). Camera temp files are stored in `cacheDir/captured/` and cleaned up on result.
- **Camera/microphone permissions**: `onPermissionRequest` checks already-granted Android runtime permissions; if missing, defers the `PermissionRequest` in `pendingWebPermissionRequest` and launches `webPermissionLauncher`, then grants/denies after the result.

# SDK / Tooling

- Kotlin, AGP 9.2.1, Gradle wrapper 9.5.1
- `compileSdk` / `targetSdk` 36, `minSdk` 24
- Version catalog: `gradle/libs.versions.toml`
