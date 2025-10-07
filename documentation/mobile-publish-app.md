# Mobile Publish App Workflow

This workflow **builds and distributes** the iOS app using Expo EAS.  
It can publish the build to **Firebase App Distribution** and/or **TestFlight**.

---

## 🔧 Inputs

- **environment** (string, default `"preview"`)  
  Build profile/environment. Options: `preview`, `production`.

- **publish_to_firebase** (boolean, default `true`)  
  Whether to publish to Firebase App Distribution.

- **publish_to_testflight** (boolean, default `true`)  
  Whether to publish to TestFlight.

- **node_version** (string, default `"23"`)  
  Node.js version.

- **bun_version** (string, default `"latest"`)  
  Bun version.

- **firebase_groups** (string, default `"internal-testers"`)  
  Firebase tester groups (comma-separated).

- **release_notes** (string, default `""`)  
  Release notes for the distribution.

---

## 🔑 Secrets

- **EXPO_TOKEN** (required)  
  Expo authentication token.

### TestFlight
Required if `publish_to_testflight` is `true`:
- `APP_STORE_CONNECT_API_ISSUER_ID`
- `APP_STORE_CONNECT_API_KEY_ID`
- `APP_STORE_CONNECT_API_KEY`

### Firebase
Required if `publish_to_firebase` is `true`:
- `PREVIEW_FIREBASE_APP_ID_IOS`
- `PRODUCTION_FIREBASE_APP_ID_IOS`
- `PREVIEW_FIREBASE_SERVICE_ACCOUNT_JSON`
- `PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON`

---

## 📌 Example Usage

```yaml
jobs:
  release:
    uses: untile/github-workflows/.github/workflows/mobile-publish-app.yml@v1
    with:
      environment: production
      publish_to_firebase: true
      publish_to_testflight: false
      node_version: "23"
      bun_version: "latest"
      firebase_groups: "qa-testers"
      release_notes: "Release build ${{ github.sha }}"
    secrets:
      EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}
      PRODUCTION_FIREBASE_APP_ID_IOS: ${{ secrets.PRODUCTION_FIREBASE_APP_ID_IOS }}
      PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON: ${{ secrets.PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON }}
      APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_ISSUER_ID }}
      APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
      APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
```

---

## 🔍 Bundle ID Validation

The workflow includes automatic bundle ID validation to ensure builds are uploaded to the correct app:

1. **After Build**: The workflow extracts and displays bundle information from the IPA:
   - Bundle ID (e.g., `com.buzzmeup.app`)
   - Bundle Name
   - Version and Build number
   - Environment being deployed

2. **Validation**: Checks that the bundle ID matches the expected value for the environment:
   - `production` environment expects: `com.buzzmeup.app`
   - `preview` environment expects: `com.buzzmeup.app.staging`

3. **Early Failure**: If there's a mismatch, the workflow fails **before** attempting to upload to TestFlight or Firebase, preventing invalid uploads.

---

## 🐛 Troubleshooting

### Bundle ID Mismatch Error

**Error**: `❌ BUNDLE ID MISMATCH! Expected: com.buzzmeup.app, Found: com.buzzmeup.app.staging`

**Cause**: The IPA was built with a different bundle identifier than expected for the environment.

**Solution**: Check your EAS build configuration in `eas.json`:

```json
{
  "build": {
    "production": {
      "ios": {
        "bundleIdentifier": "com.buzzmeup.app"
      }
    },
    "preview": {
      "ios": {
        "bundleIdentifier": "com.buzzmeup.app.staging"
      }
    }
  }
}
```

Ensure the `bundleIdentifier` matches the expected value for each profile.

---

### TestFlight Upload Error: Bundle Identifier Cannot Be Changed

**Error**: `90055: This bundle is invalid. The bundle identifier cannot be changed from the current value, 'com.buzzmeup.app.staging'.`

**Causes**:
1. **Wrong IPA**: The IPA contains a different bundle ID than expected (caught by validation above)
2. **API Key Permissions**: The App Store Connect API key doesn't have access to the target app
3. **App Not Created**: The app with the target bundle ID doesn't exist in App Store Connect

**Solutions**:
1. Check the bundle ID validation output in the workflow logs
2. Verify the API key has access to both apps in App Store Connect (Users and Access → Keys)
3. Ensure both apps exist in App Store Connect with the correct bundle IDs
4. Consider using separate API keys for preview vs production environments

---

### Firebase Distribution Error

**Error**: `❌ FIREBASE_CREDENTIALS is empty`

**Solution**: Ensure the correct Firebase secrets are configured:
- For `environment: production`: Set `PRODUCTION_FIREBASE_APP_ID_IOS` and `PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON`
- For `environment: preview`: Set `PREVIEW_FIREBASE_APP_ID_IOS` and `PREVIEW_FIREBASE_SERVICE_ACCOUNT_JSON`
