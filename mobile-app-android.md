# Android app pattern: Kotlin + GitHub Actions + in-app update

A reusable pattern for personal Android apps. Build is CI-only (no local Android Studio). The app downloads and installs its own updates from GitHub Releases.

Used by: `musical-succotash` (Flutter variant), `birdscope` (Kotlin native variant).

## Stack

- Kotlin native (Android-only)
- AGP 8.5.x, Kotlin 2.0.x, Gradle 8.7
- `compileSdk 34`, `minSdk 24`, `targetSdk 34`
- ViewBinding (no Compose, no DataBinding)
- AppCompat + Material3 themes
- `kotlinx-coroutines-android` for async work

## Repo layout

```
.
├── .github/workflows/build.yml
├── build.gradle              (root, plugins only)
├── settings.gradle           (rootProject.name + include :app)
├── gradle.properties
└── app/
    ├── build.gradle
    ├── dev-release.jks       (committed dev signing key — see below)
    └── src/main/
        ├── AndroidManifest.xml
        ├── kotlin/<package>/
        │   ├── MainActivity.kt
        │   ├── Updater.kt
        │   └── InstallStatusReceiver.kt
        └── res/
            ├── drawable/ic_launcher.xml   (vector, no PNGs needed)
            ├── layout/activity_main.xml
            └── values/{strings,themes}.xml
```

No gradle wrapper checked in. CI uses `gradle/actions/setup-gradle@v4` with `gradle-version: "8.7"`.

## Signing key

- One dev keystore (`dev-release.jks`) is committed to the repo.
- Same passwords across all projects (e.g. `hexcanvas`/`devkey`/`hexcanvas`) so the keystore can be reused.
- **Why commit it:** the in-app updater requires the same signing key on every build, otherwise Android refuses the update with "signature mismatch". A keystore generated fresh per CI run breaks updates.
- Generate once with `keytool`, commit, reuse forever:
  ```
  keytool -genkeypair -v -keystore dev-release.jks -keyalg RSA -keysize 2048 \
    -validity 10000 -alias devkey -storepass <pass> -keypass <pass> \
    -dname "CN=Dev"
  ```
- This is a personal-use key; do **not** publish to the Play Store with it.

## CI workflow (`.github/workflows/build.yml`)

```yaml
name: Build
on:
  push:
    branches: ["**"]
    paths-ignore: ['**.md']
  workflow_dispatch:

jobs:
  android:
    runs-on: ubuntu-latest
    permissions:
      contents: write   # needed for `gh release create`
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: "17" }
      - uses: gradle/actions/setup-gradle@v4
        with: { gradle-version: "8.7" }

      - env: { BUILD_NUMBER: ${{ github.run_number }} }
        run: gradle :app:assembleRelease --no-daemon

      - run: cp app/build/outputs/apk/release/app-release.apk <app>.apk

      - uses: actions/upload-artifact@v4
        with: { name: <app>-apk, path: <app>.apk, retention-days: 30 }

      - if: github.ref == 'refs/heads/main'
        env: { GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} }
        run: |
          gh release create "build-${{ github.run_number }}" \
            --title "Build ${{ github.run_number }}" \
            --notes "Commit: ${{ github.sha }}" \
            --latest \
            <app>.apk
```

`versionCode = BUILD_NUMBER` in `app/build.gradle`. CI's `github.run_number` is monotonic and per-repo, so it works as a stable, ever-increasing build number.

## In-app updater

Three pieces.

### 1. `Updater.kt` (Kotlin)

- Reads installed `versionCode` from `PackageManager`.
- Hits `https://github.com/<owner>/<repo>/releases/latest` with `instanceFollowRedirects = false`. The 302 `Location` header points at `/releases/tag/build-N`. Parses N.
  - **Why not the API:** GitHub API has a 60 req/hour anonymous rate limit which carrier NAT hits fast. The web redirect has no such limit.
- If `latest > installed`: downloads `<repo-base>/releases/download/build-N/<app>.apk` via Android `DownloadManager` to `Downloads/<app>_update.apk`.
- Polls download status, then calls `installApk()`.

### 2. `installApk()` — PackageInstaller session API

- Open a `PackageInstaller` session, write the APK bytes into the session, commit with a `PendingIntent` for the result broadcast.
- Do **not** use `Intent.ACTION_INSTALL_PACKAGE` with a `content://` URI from a FileProvider — it works but is fragile across OEMs and can fail with "exposed beyond app" / parse errors.
- The broadcast comes back with `STATUS_PENDING_USER_ACTION`; a `BroadcastReceiver` extracts the contained `Intent.EXTRA_INTENT` and starts it — that is the actual "Do you want to install?" system dialog.

### 3. Permission gate

If `packageManager.canRequestPackageInstalls()` is false (first run on Android 8+), open `Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES` for the app, then ask the user to retry the update.

### Manifest needs

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />

<receiver android:name=".InstallStatusReceiver" android:exported="false">
  <intent-filter>
    <action android:name="<package>.INSTALL_STATUS" />
  </intent-filter>
</receiver>
```

## First install

In-app updater can't install the very first APK (no app present yet). User must:
1. Open `https://github.com/<owner>/<repo>/releases/latest` in a phone browser.
2. Download the APK, enable "install from unknown sources" if prompted, install.
3. After that, the in-app "Check for update" button handles every later version.

## Tagging

Per-phase tags (e.g. `phase-1`, `phase-2`) on milestone commits. CI also creates `build-N` release tags automatically; those are runtime artifacts, not phase markers.

## Common gotchas

- **Keystore password mismatch** — if reusing a keystore from another project, copy the exact passwords from the old `app/build.gradle`. Build fails with "keystore password was incorrect".
- **AGP 8+ disables BuildConfig by default.** If you reference `BuildConfig.VERSION_NAME`, add `buildFeatures { buildConfig true }` — or just read `packageInfo.versionName` directly.
- **No mipmap launcher icons.** A single vector drawable in `res/drawable/ic_launcher.xml` referenced from the manifest is enough on `minSdk 24`.
- **CRLF line endings on Windows.** Git warns about LF→CRLF conversion. Harmless for build; ignore.
