# Android modernization

This fork preserves Hacker's Keyboard's legacy behavior while updating the Android build and runtime compatibility for current Android releases.

## Toolchain

- Android Gradle Plugin: 9.3.0
- Gradle: 9.5.0
- Java source/target: 17
- compileSdk: 36
- targetSdk: 36
- minSdk: 21
- NDK: 29.0.14206865
- CMake: 3.31.6

`minSdk` was raised from 14 to 21 because the current NDK used for the native dictionary no longer supports API levels below 21.

## Compatibility changes

- Removed the obsolete Android Support notification dependency and use platform notification APIs.
- Added immutable `PendingIntent` flags required by current Android versions.
- Added Android 13+ dynamic broadcast receiver flags while retaining the pre-33 path.
- Added explicit component export state for Android 12+.
- Added package visibility queries for external dictionary plugins.
- Added Android 15+ navigation-bar inset handling around the IME so system hide/switch controls do not overlap bottom-row keys; side navigation insets are also respected in landscape.
- The gesture-area spacer follows the active keyboard theme, including translucent ARGB themes, without double-compositing the keyboard itself.
- IME safe-inset application now caches the last non-zero bottom inset and re-applies root insets after view attachment, preventing transient zero-inset layouts when switching IMEs or recreating a theme; the system navigation divider/contrast layer is disabled so the themed gesture area remains seamless.
- The gesture-safe area and long-press mini-keyboard use a borderless fill derived from the active keyboard background (exact ARGB for color themes, center-sampled color for 9-patches), avoiding both legacy black popup panels and visible 9-patch edge seams.
- Routed KeyboardSwitcher input-view refreshes through the same inset-aware container to avoid re-parenting crashes when Android rebinds the IME.
- On gesture-navigation devices, the IME safe bottom area now uses the maximum of navigation-bar, mandatory-system-gesture, and tappable-element insets; some Android 16 IME windows report navigationBars.bottom as zero even while reserving the system hide/switch strip.
- Physical Android 16 verification on Xiaomi 21121210G measured navBottom=0, mandatoryBottom=119, tappableBottom=119, so the IME applies a 119 px bottom safe spacer.
- Physical Android 16 regression test for `modern.8`: with the built-in Test editor focused, 12 Gboard ↔ Hacker's Keyboard switches produced 45 inset applications, all with `appliedBottom=119` and zero with `appliedBottom=0`; the IME remained shown and stable.
- Physical Android 16 visual regression test for `modern.10`: the lower keyboard/safe-area boundary and all rows through y=2399 remained continuously Material Dark `#263238` with no bright seam; an `e` long-press popup used the same `#263238` theme fill (zero pure-black changed pixels) while the underlying keyboard dimmed normally.
- Updated resource-ID handling for modern non-final `R` values.
- Updated CMake project metadata for the current native toolchain.
- Preserved the upstream legacy keyboard resource namespaces and incomplete translation set; the matching legacy-only lint checks are disabled rather than generating a very large no-behavior resource diff.

## Verification

Run from the repository root with Android Studio's JBR/JDK 17+ available:

```sh
./gradlew :app:lintDebug :app:assembleDebug :app:assembleRelease
```

Verified during the modernization:

- `lintDebug`: 0 errors
- debug APK builds and verifies successfully
- unsigned release APK builds successfully
- APK declares `targetSdkVersion 36` and `sdkVersion 21`
- native library builds for arm64-v8a, armeabi-v7a, x86, and x86_64
- debug APK passes 16 KiB ZIP alignment verification
- `v1.41.1-modern.3` was installed successfully with Gradle `installDebug` on the physical Xiaomi 21121210G running Android 16 / API 36; visual confirmation of the navigation-control overlap fix is pending.

Physical-device verification now includes a Xiaomi 21121210G (`ingres`) running Android 16 / API 36 over wireless ADB. The IME service starts and loads its arm64 native library successfully. The settings regression test covers Theme and font, Input languages, and Key feedback; these internal screens use explicit component intents so they remain reachable while their activities stay non-exported.


Physical Android 16 testing is now performed on a Xiaomi 21121210G (`ingres`). The `modern.4` regression test selected Hacker's Keyboard, focused a real text field, confirmed `mInputShown=true` / `mIsInputViewShown=true`, and observed no fresh `FATAL EXCEPTION`; Gboard was restored after the test.
