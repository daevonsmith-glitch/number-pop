# Number Pop 🫧

Pop the bubble with the right answer. Addition, subtraction, multiplication and division, or mix them all. Made for kids and anyone who stims: no timer, no penalty for wrong taps, color-changing bubbles, counting pictures, and every problem and answer is spoken out loud ("3 plus 4 equals 7!"). Plays on phones and tablets: Android phones and tablets, iPhone and iPad.

- `www/` – the game itself (this is what goes on Cloudflare)
- `www/privacy.html` – the privacy policy page (fill in the two blanks before release)
- `.github/workflows/android-apk.yml` – builds the Android APK and the Google Play file on GitHub
- `.github/workflows/ios-build.yml` – builds the iPhone/iPad app on GitHub and sends it to App Store Connect
- `package.json`, `capacitor.config.json` – settings the builds use
- `assets/icon-1024.png` – the logo used for the app icons
- `STORE-GUIDE.md` – step by step for Google Play and the App Store, with listing text

---

## 1. Put it on GitHub

1. Go to github.com → **+** → **New repository**. Name it `number-pop`. Make it **Public** (so the APK download link works for anyone). Tick **Add a README file**, then **Create repository**.
2. Upload the game: **Add file → Upload files**, choose **`number-pop.zip`** (the zip itself, not unzipped), and **Commit changes**.
3. Add the Android build file: **Add file → Create new file**. In the name box type exactly `.github/workflows/android-apk.yml`, paste in the Android build file from the bottom of this guide, and **Commit changes**.
4. Add the iPhone/iPad build file the same way, named `.github/workflows/ios-build.yml`.

On a phone, do this in Safari or Chrome, not the GitHub app. On a computer you can instead unzip the package and drag in everything inside the `number-pop` folder, including the hidden `.github` folder.

## 2. Get the Android APK

1. In the repo, open the **Actions** tab. "Build Android APK" starts by itself (about 5–8 minutes). If Actions asks you to enable workflows, tap enable.
2. When it shows a green check, go to the repo's **Releases** → **Number Pop (latest build)** → tap **NumberPop.apk**.
3. Open the downloaded file on the Android phone or tablet. Android will ask to allow installs from your browser or files app. Allow it, then **Install**.

Every time you change a file on `main`, a fresh APK is built and replaces the old one on the same Releases page. For the Google Play file (`NumberPop.aab`), see `STORE-GUIDE.md` step 1B.

If a build fails: **Actions** → the failed run → **build** → the step with the red ❌. The red lines near the bottom of that step say what went wrong.

## 3. Put it on Cloudflare (iPhone, iPad + any browser)

1. Sign in at dash.cloudflare.com → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**.
2. Pick the `number-pop` GitHub repo.
3. Build settings:
   - Framework preset: **None**
   - Build command: *(leave empty)*
   - Build output directory: **`www`**
4. **Save and Deploy.** You get a link like `https://number-pop.pages.dev`. Share that.

Do this after the first GitHub build has finished, so the `www` folder is in the repository. Cloudflare redeploys automatically whenever the GitHub repo changes.

### Install on iPhone or iPad
Open the Cloudflare link in **Safari** → **Share** → **Add to Home Screen**. It opens full-screen like an app and works offline after the first visit.

Tip: if numbers aren't read aloud on iPhone, check that the silent switch is off.

---

## Updating the game
Edit `www/index.html`. Also change `VERSION` in `www/sw.js` (for example to `number-pop-v5`) so installed phones pick up the change.

---

## The Android build file

Paste this as `.github/workflows/android-apk.yml`:

```yaml
name: Build Android APK

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-24.04
    env:
      KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
      KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    steps:
      - uses: actions/checkout@v4

      # Lets you upload the whole game as one zip from a phone: this unpacks it into the repository.
      - name: Unpack number-pop.zip
        run: |
          if [ -f number-pop.zip ]; then
            unzip -q -o number-pop.zip -d /tmp/np
            rm -rf /tmp/np/number-pop/.github
            cp -r /tmp/np/number-pop/. .
            git rm -q number-pop.zip
            git add -A
            git config user.name "github-actions[bot]"
            git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
            git commit -q -m "Unpack game files from number-pop.zip"
            git push
          fi

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Install Capacitor
        run: npm install --no-audit --no-fund --legacy-peer-deps

      - name: Create Android project
        run: npx cap add android

      - name: Copy the game into the app
        run: npx cap sync android

      - name: Allow access to the phone's speech engine
        run: |
          M=android/app/src/main/AndroidManifest.xml
          grep -q TTS_SERVICE "$M" || sed -i 's#</manifest>#    <queries>\n        <intent>\n            <action android:name="android.intent.action.TTS_SERVICE" />\n        </intent>\n    </queries>\n</manifest>#' "$M"

      # Google Play needs a higher version number for every upload, so use the build number.
      - name: Set version number
        run: |
          G=android/app/build.gradle
          sed -i -E "s/versionCode [0-9]+/versionCode ${{ github.run_number }}/; s/versionName \"[^\"]*\"/versionName \"1.0.${{ github.run_number }}\"/" "$G"
          grep -E "versionCode|versionName" "$G"

      - name: Add app icon
        continue-on-error: true
        run: |
          R=android/app/src/main/res
          for d in mdpi:48 hdpi:72 xhdpi:96 xxhdpi:144 xxxhdpi:192; do
            n=${d%%:*}; s=${d##*:}
            mkdir -p $R/mipmap-$n
            convert assets/icon-1024.png -resize ${s}x${s} $R/mipmap-$n/ic_launcher.png
            cp $R/mipmap-$n/ic_launcher.png $R/mipmap-$n/ic_launcher_round.png
            rm -f $R/mipmap-$n/ic_launcher_foreground.png
          done
          rm -rf $R/mipmap-anydpi-v26

      - name: Build APK (for sharing directly)
        working-directory: android
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon --stacktrace

      - name: Rename APK
        run: cp android/app/build/outputs/apk/debug/app-debug.apk NumberPop.apk

      # The Play Store file. Only runs once the two signing secrets are saved in the repository settings.
      - name: Build and sign the Play Store bundle
        if: env.KEYSTORE_BASE64 != ''
        continue-on-error: true
        run: |
          (cd android && ./gradlew bundleRelease --no-daemon --stacktrace)
          A=android/app/build/outputs/bundle/release/app-release.aab
          echo "$KEYSTORE_BASE64" | base64 -d > "$RUNNER_TEMP/upload.jks"
          jarsigner -keystore "$RUNNER_TEMP/upload.jks" -storepass "$KEYSTORE_PASSWORD" -keypass "$KEYSTORE_PASSWORD" \
            -sigalg SHA256withRSA -digestalg SHA-256 "$A" upload
          rm -f "$RUNNER_TEMP/upload.jks"
          cp "$A" NumberPop.aab

      - name: Keep a copy on the run page
        uses: actions/upload-artifact@v4
        with:
          name: NumberPop
          path: |
            NumberPop.apk
            NumberPop.aab
          if-no-files-found: warn

      - name: Publish to Releases
        uses: softprops/action-gh-release@v2
        with:
          tag_name: latest
          name: Number Pop (latest build)
          body: |
            NumberPop.apk - open it on an Android phone or tablet to install the game directly.
            NumberPop.aab - the file to upload to the Google Play Console (only present once signing is set up).
          files: |
            NumberPop.apk
            NumberPop.aab
          fail_on_unmatched_files: false
          make_latest: true
```

## The iPhone/iPad build file

Paste this as `.github/workflows/ios-build.yml`:

```yaml
name: Build iPhone and iPad app

# Run it by hand: Actions -> Build iPhone and iPad app -> Run workflow.
# It builds the game for iPhone and iPad, signs it with your Apple account and sends it to App Store Connect (TestFlight).
on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  ios:
    runs-on: macos-26
    env:
      ASC_KEY_ID: ${{ secrets.ASC_KEY_ID }}
      ASC_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
      ASC_KEY_P8: ${{ secrets.ASC_KEY_P8 }}
      APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
    steps:
      - uses: actions/checkout@v4

      - name: Check the four Apple secrets are saved
        run: |
          for v in ASC_KEY_ID ASC_ISSUER_ID ASC_KEY_P8 APPLE_TEAM_ID; do
            if [ -z "${!v}" ]; then echo "::error::Missing repository secret $v (see STORE-GUIDE.md, step 2C)"; exit 1; fi
          done

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install Capacitor
        run: npm install --no-audit --no-fund --legacy-peer-deps

      - name: Create iPhone/iPad project
        run: npx cap add ios

      - name: Copy the game into the app
        run: npx cap sync ios

      - name: App settings
        run: |
          P=ios/App/App/Info.plist
          # No encryption, so App Store Connect won't ask about export rules for each build.
          /usr/libexec/PlistBuddy -c "Add :ITSAppUsesNonExemptEncryption bool false" "$P" || /usr/libexec/PlistBuddy -c "Set :ITSAppUsesNonExemptEncryption false" "$P"
          /usr/libexec/PlistBuddy -c "Set :CFBundleDisplayName Number Pop" "$P" || /usr/libexec/PlistBuddy -c "Add :CFBundleDisplayName string Number Pop" "$P"
          # iPad: allow every orientation (Apple requires this for iPad multitasking).
          /usr/libexec/PlistBuddy -c "Delete :UISupportedInterfaceOrientations~ipad" "$P" || true
          /usr/libexec/PlistBuddy -c "Add :UISupportedInterfaceOrientations~ipad array" "$P"
          for o in Portrait PortraitUpsideDown LandscapeLeft LandscapeRight; do
            /usr/libexec/PlistBuddy -c "Add :UISupportedInterfaceOrientations~ipad: string UIInterfaceOrientation$o" "$P"
          done
          plutil -p "$P"

      - name: Add app icon
        run: |
          I=$(ls ios/App/App/Assets.xcassets/AppIcon.appiconset/*.png | head -1)
          cp assets/icon-1024.png "$I"
          echo "Icon written to $I"

      - name: Save the App Store Connect key
        run: |
          mkdir -p ~/.appstoreconnect/private_keys
          printf '%s\n' "$ASC_KEY_P8" > ~/.appstoreconnect/private_keys/AuthKey_$ASC_KEY_ID.p8

      # TARGETED_DEVICE_FAMILY 1,2 = iPhone and iPad
      - name: Build
        run: |
          xcodebuild archive \
            -project ios/App/App.xcodeproj -scheme App -configuration Release \
            -destination 'generic/platform=iOS' -archivePath "$RUNNER_TEMP/App.xcarchive" \
            MARKETING_VERSION=1.0 CURRENT_PROJECT_VERSION=${{ github.run_number }} \
            TARGETED_DEVICE_FAMILY="1,2" DEVELOPMENT_TEAM="$APPLE_TEAM_ID" \
            CODE_SIGNING_ALLOWED=NO

      - name: Sign for the App Store
        run: |
          cat > "$RUNNER_TEMP/ExportOptions.plist" <<PLIST
          <?xml version="1.0" encoding="UTF-8"?>
          <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
          <plist version="1.0">
          <dict>
            <key>method</key><string>app-store-connect</string>
            <key>signingStyle</key><string>automatic</string>
            <key>teamID</key><string>$APPLE_TEAM_ID</string>
            <key>destination</key><string>export</string>
            <key>uploadSymbols</key><true/>
          </dict>
          </plist>
          PLIST
          xcodebuild -exportArchive \
            -archivePath "$RUNNER_TEMP/App.xcarchive" \
            -exportPath "$RUNNER_TEMP/export" \
            -exportOptionsPlist "$RUNNER_TEMP/ExportOptions.plist" \
            -allowProvisioningUpdates \
            -authenticationKeyPath ~/.appstoreconnect/private_keys/AuthKey_$ASC_KEY_ID.p8 \
            -authenticationKeyID "$ASC_KEY_ID" \
            -authenticationKeyIssuerID "$ASC_ISSUER_ID"
          cp "$RUNNER_TEMP"/export/*.ipa NumberPop.ipa

      - name: Send to App Store Connect
        run: xcrun altool --upload-app -f NumberPop.ipa -t ios --apiKey "$ASC_KEY_ID" --apiIssuer "$ASC_ISSUER_ID"

      - name: Keep a copy on the run page
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: NumberPop-ios
          path: NumberPop.ipa
          if-no-files-found: ignore

      - name: Remove the key from the build machine
        if: always()
        run: rm -rf ~/.appstoreconnect
```
