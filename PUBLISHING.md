# Publishing Harika Millets Powders Android App to Google Play

This document describes how to produce a signed Android App Bundle (AAB) and publish it to the Google Play Console using the GitHub Actions workflow included in `.github/workflows/android-release.yml`.

Overview
- The repository contains a Capacitor Android project under `android/` and a GitHub Actions workflow that builds the web app, copies it into the Android project, and runs `./gradlew bundleRelease` to produce an AAB.
- To publish automatically, the workflow needs:
  - A keystore (base64-encoded) provided as the `KEYSTORE_BASE64` secret, and the keystore passwords and alias as secrets
  - A Google Play service account JSON placed in the `PLAY_SERVICE_ACCOUNT_JSON` secret (optional — required only for automatic publish)

Important: Never commit your keystore or Play service account JSON to the repository. Use GitHub Actions secrets.

## 1) Create a release keystore (on your machine)

Run this (PowerShell) to generate a keystore using `keytool` (part of JDK):

```powershell
keytool -genkeypair -v -keystore my-release-key.jks -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

Follow the prompts and remember:
- Keystore password (STORE_PASSWORD)
- Key alias (KEY_ALIAS) — e.g. `my-key-alias`
- Key password (KEY_PASSWORD)

## 2) Base64-encode the keystore for GitHub Actions

PowerShell command (writes base64 to clipboard):

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes('my-release-key.jks')) | Set-Clipboard
# Now paste into the GitHub secret named KEYSTORE_BASE64
```

Or save to a file:

```powershell
$b = [Convert]::ToBase64String([IO.File]::ReadAllBytes('my-release-key.jks'))
$b | Out-File -Encoding ascii keystore-base64.txt
```

## 3) Create Google Play service account (optional — for automatic publish)

- Go to Google Play Console → Settings → API access → Create Service Account
- Grant the service account appropriate release permissions
- Download the JSON key and copy its contents into the `PLAY_SERVICE_ACCOUNT_JSON` secret (GitHub repo secrets)

## 4) Add GitHub secrets

Repository → Settings → Secrets and variables → Actions → New repository secret

- `KEYSTORE_BASE64`: content of base64-encoded keystore (or leave empty to sign locally)
- `STORE_PASSWORD`: keystore password
- `KEY_ALIAS`: alias used when creating the key
- `KEY_PASSWORD`: key password
- `PLAY_SERVICE_ACCOUNT_JSON` (optional): entire JSON contents

## 5) Push to `main` branch

The workflow will run on push to `main`, build the app, and produce `app-release.aab` as an artifact. If `PLAY_SERVICE_ACCOUNT_JSON` is set, it will attempt to publish to the `internal` track.

## 6) Manual Publish (if you prefer not to use automation)

Build locally after installing Java + Android SDK (Android Studio):

```powershell
npm run build
npx cap copy android
cd android
.\gradlew.bat bundleRelease
# Resulting AAB: android\app\build\outputs\bundle\release\app-release.aab
```

Then upload the generated AAB to Google Play Console manually.

## 7) If you want me to push this repo to GitHub and set up secrets, tell me the repository name (I will create the remote and push). I cannot set secrets for you — these must be added from your GitHub account.

## Troubleshooting
- If `./gradlew` fails locally with `JAVA_HOME` not set: install JDK and set `JAVA_HOME` environment variable.
- If the GitHub workflow fails: check Actions logs; common issues are missing secrets or an incorrect base64 keystore.

## Security notes
- Keep your keystore backups secure. Losing the keystore will prevent you from updating the app on Play Store under the same package name.
