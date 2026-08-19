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

A physical-device smoke test was not run at the time of this commit because no Android device was connected over ADB.
