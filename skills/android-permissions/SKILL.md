---
name: android-permissions
description: Diagnose and fix Android permission problems in Flutter or native Android projects. Use when a permission is missing from Android settings, a runtime request does not appear, a feature reports permission denied, or behavior differs between source code, the merged manifest, the final APK/AAB, and the installed application.
license: MIT
metadata:
  author: bdssmdkacem-dot
  version: "1.0.0"
  category: mobile-engineering
---

# Android Permissions

Use this skill for permission-related bugs in Flutter/Android applications.

The core rule is:

> Never assume a permission problem is fixed because the Dart/Kotlin/Java source or a build succeeded. Verify the complete chain from source declaration to the installed application.

## 1. Establish the exact symptom

Record:

- application/package ID
- Android version/API level
- build type (debug/profile/release)
- APK or AAB versionCode/versionName when available
- exact permission expected
- whether the problem is:
  - permission absent from Settings
  - permission dialog never appears
  - permission is denied
  - permission is granted but the feature still fails
  - behavior changes after reinstall/update
- whether the app was upgraded over an older installation

Do not modify code before establishing which layer is failing, unless the source already proves the defect.

## 2. Trace the permission through every layer

Check, in order:

1. Flutter/native source requesting the permission.
2. AndroidManifest.xml files in the application and relevant libraries.
3. Gradle configuration and Android SDK/target configuration.
4. Merged manifest produced by the Android build.
5. Final APK/AAB manifest.
6. Runtime permission state on the installed package.
7. Android system settings and relevant logs.

Android requires protected permissions to be declared in the manifest, while dangerous/runtime permissions also require runtime approval where applicable. A manifest declaration and a runtime grant are separate facts.

## 3. Flutter-specific checks

Identify the package responsible for the permission, for example:

- geolocator
- permission_handler
- flutter_local_notifications
- bluetooth-related plugins
- camera/microphone plugins
- activity-recognition plugins

Check both:

- the application's own Android manifest
- manifest contributions from dependencies

Do not add duplicate or unrelated permissions merely to make a feature work.

For a Flutter permission API, verify:

- the request code is actually executed
- the request is awaited where required
- the returned status is handled
- permanently denied/restricted states are handled appropriately
- the Android implementation matches the feature's required access level

## 4. Android manifest verification

Search all relevant manifests for the exact permission.

For example:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

Do not conclude that source declaration is sufficient.

The merged manifest is the authoritative build-time result. When possible, inspect the final APK/AAB as well.

## 5. Runtime verification

If ADB is available, inspect the installed package rather than guessing from the UI.

Useful checks include:

```bash
adb shell pm list packages | grep <package>
adb shell dumpsys package <package>
adb shell cmd appops get <package>
```

On Windows PowerShell, use:

```powershell
adb shell pm list packages | Select-String "<package>"
adb shell dumpsys package <package>
adb shell cmd appops get <package>
```

Use the package ID actually found in the project. Never invent it.

When ADB is unavailable, explicitly distinguish:

- evidence from source/build artifacts
- evidence from the Android Settings UI
- evidence that has not yet been verified

## 6. Clean-install and upgrade behavior

Permission state can differ between:

- fresh install
- upgrade over an existing installation
- uninstall/reinstall

When the symptom concerns a permission that was previously denied or absent, test both:

1. upgrade installation
2. clean installation after uninstalling the package

Do not tell the user that reinstalling is the root cause unless the evidence supports that conclusion.

## 7. Android version differences

Check the project's target SDK and the device API level before applying a permission rule.

Do not copy an old Android permission recipe blindly. Android permission behavior changes across API levels.

For location, distinguish:

- approximate location
- precise location
- foreground location
- background location

Request only the access required by the actual feature.

## 8. Dependency permissions

A dependency may contribute manifest entries or require additional runtime handling.

If a permission appears unexpectedly:

1. identify which dependency contributes it
2. determine why it is present
3. remove or restrict it only if safe and supported

Do not remove a manifest entry blindly.

## 9. Diagnose before fixing

Classify the defect into one or more layers:

- SOURCE
- MANIFEST
- MERGED_MANIFEST
- ARTIFACT
- RUNTIME
- OS_SETTINGS
- DEVICE/API
- DEPENDENCY
- BUILD_CONFIGURATION

Then make the smallest change that addresses the proven failing layer.

Avoid unrelated refactors.

## 10. Verification after a code change

After modifying a permission-related implementation:

1. run formatter if appropriate
2. run static analysis
3. run relevant tests
4. build the affected Android artifact
5. inspect the resulting artifact's manifest when possible
6. install the artifact when a device is available
7. verify the runtime permission state
8. test the affected feature
9. compare the result with the original symptom

A successful CI build is not sufficient evidence that the permission problem is resolved.

## 11. Reporting

When reporting the result, state:

- what was checked
- the first layer where the defect was proven
- what changed
- what was verified
- what remains unverified
- the exact artifact/build used for verification

Use precise language.

Good:

> ACCESS_FINE_LOCATION and ACCESS_COARSE_LOCATION are present in the final APK manifest. Runtime grant state was then checked on the installed package. The feature was verified on Android 13.

Avoid:

> GPS is definitely fixed.

unless the actual device behavior was tested.

## 12. Safety and scope

Never:

- request unnecessary sensitive permissions
- weaken Android security controls
- bypass runtime consent
- add broad permissions without a documented feature requirement
- modify signing configuration merely because a permission is failing
- delete existing working configuration without evidence

Preserve unrelated application behavior.

## References

For detailed artifact inspection procedures, read:

- `references/apk-verification.md`

Use the reference only when artifact-level verification is needed.
