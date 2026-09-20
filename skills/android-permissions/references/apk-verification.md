# APK Permission Verification

## Purpose

Use this reference when source and manifest inspection are insufficient and the final APK must be verified.

## Verification chain

```
Flutter/native source
    |
    v
AndroidManifest.xml
    |
    v
Merged Manifest
    |
    v
APK/AAB
    |
    v
Installed package
    |
    v
Runtime permission/app-op state
```

## APK inspection

If Android build tools are available, locate the final APK and inspect its manifest.

A common command is:

```bash
apkanalyzer manifest permissions path/to/app-release.apk
```

If `apkanalyzer` is unavailable, use an equivalent Android SDK/build-tools inspection method. Do not claim the APK was inspected when it was not.

For lower-level inspection, `aapt2` can be used where available:

```bash
aapt2 dump permissions path/to/app-release.apk
```

The exact tool path depends on the installed Android SDK/build-tools version.

## Runtime package inspection

With a connected device:

```bash
adb shell dumpsys package <package-id>
adb shell cmd appops get <package-id>
```

Look for the requested permission and its current grant/operation state.

Runtime evidence must be tied to the exact installed package/version being tested.

## Manifest interpretation

A permission appearing in the source manifest proves only that the source declares it.

A permission appearing in the merged manifest proves that the Android build system retained it.

A permission appearing in the final APK proves that the shipped artifact contains it.

A runtime grant proves that the installed application has received the corresponding access.

These are different verification levels and should not be conflated.

## Location example

For location features, verify the exact requirements.

Typical foreground location permissions are:

```text
android.permission.ACCESS_COARSE_LOCATION
android.permission.ACCESS_FINE_LOCATION
```

If background access is actually required, verify the additional Android requirements instead of automatically adding background location.

## Clean installation

For permission-state bugs, record whether the test is:

- fresh install
- upgrade
- uninstall + reinstall

If the behavior differs, report that difference explicitly.

## Evidence table

When diagnosing a difficult issue, record:

| Layer | Evidence | Result |
|---|---|---|
| Source | manifest/request code | pass/fail |
| Merged manifest | build output | pass/fail |
| APK | final artifact | pass/fail |
| Runtime | dumpsys/appops | pass/fail |
| Settings | device UI | pass/fail |
| Feature | real behavior | pass/fail |

The final conclusion should be based on the strongest available evidence, and missing evidence should be called out rather than inferred.
