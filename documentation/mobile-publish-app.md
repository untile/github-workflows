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
