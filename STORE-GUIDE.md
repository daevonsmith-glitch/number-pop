# Number Pop Kids – step by step for Google Play and the App Store

How to build Number Pop Kids for both stores and publish it under your own name (personal accounts). It works on phones and tablets: Android phones and tablets, iPhone and iPad.

**What publishing as a person means:** no D-U-N-S number is needed, but your legal name is shown publicly as the seller on both stores, and Google makes every new game run a 14-day closed test with at least 12 testers before it can go public.

**What's already done:** the game (phone and tablet layouts), the new logo, the Android build (APK + Play Store file), the iPhone/iPad build, Number Pop Kids's Google signing key, the privacy policy page, the store text below, and every store graphic and screenshot.

**Kids' game rules:** Number Pop Kids is made for children, so Google's **Families** rules apply, and on Apple you can list it in the **Kids** category. The game already meets the main rules: no ads, no purchases, no accounts, no links out of the game, no tracking, and it doesn't load anything from other websites.

---

## Part 0. Before either store (do once)

1. **Fill in the privacy policy.** In `www/privacy.html`, replace `[YOUR NAME]` with your name and `[YOUR CONTACT EMAIL]` with an email parents can write to.
2. **Decide the app ID.** It is set to `com.numberpop.game` in `capacitor.config.json`. Both stores use it as the game's permanent ID, and it **can never be changed** after the first upload. Keep it, or change it now (for example `com.yourname.numberpop`).
3. **Put the game on GitHub.**
   1. github.com → **+** → **New repository**. Name it `number-pop`, make it **Public**, tick **Add a README file**, **Create repository**.
   2. **Add file → Upload files**, choose **`number-pop.zip`** (the zip itself, don't unzip it), **Commit changes**.
   3. **Add file → Create new file**, name it exactly `.github/workflows/android-apk.yml`, paste the Android build file from the end of `README.md`, **Commit changes**. The first build unpacks the zip for you.
   4. **Add file → Create new file** again, name it `.github/workflows/ios-build.yml`, paste the iPhone/iPad build file from the end of `README.md`, **Commit changes**.

   On a computer you can instead unzip and drag in everything inside the `number-pop` folder, including the hidden `.github` folder.
4. **Put it on Cloudflare** (README section 3). Both stores need a public privacy policy link, which will be `https://YOUR-CLOUDFLARE-LINK/privacy.html`.

---

## Part 1. Google Play

### 1A. Open the account (once, shared by all your games)

1. Go to **play.google.com/console** and sign in with the Google account that should own the games. You must be 18 or older.
2. Choose **Personal** as the account type.
3. Enter a developer name (what players see), your legal name, address, email and phone. Verify the email and phone with the codes Google sends.
4. Pay the **$25** fee. It is charged once, with no renewal.
5. Upload a government photo ID when asked.
6. If asked, confirm you have a real Android phone by signing in to the **Play Console** app on it.

### 1B. Build the Play Store file

1. Open **number-pop-play-store-signing-key.txt** (sent separately, never put it in GitHub).
2. In the `number-pop` GitHub repository: **Settings → Secrets and variables → Actions → New repository secret**.
3. Add `KEYSTORE_PASSWORD` with the password from the file.
4. Add `KEYSTORE_BASE64` with the long single line from the file.
5. **Actions → Build Android APK → Run workflow**.
6. After about 5 to 8 minutes, open **Releases → Number Pop (latest build)**. You get two files:
   - `NumberPop.apk` – install it on your own Android phone or tablet to test.
   - `NumberPop.aab` – the file you upload to Google Play.

Every run gives the game a higher version number automatically, which Google requires for each upload.

### 1C. Create the app and listing

1. In Play Console: **Create app** → name `Number Pop Kids`, language English (United States), **Game**, **Free**. Tick the declarations and **Create app**.
2. **Store settings → App category:** Game → **Educational**.
3. **Main store listing:** paste the text from "Store text" below, then upload from the `google` folder in **number-pop-store-graphics.zip**:

| Slot | File |
|---|---|
| App icon | `app-icon-512.png` |
| Feature graphic | `feature-graphic-1024x500.png` |
| Phone screenshots | `phone-1-menu.png` to `phone-4-subtraction-correct.png` |
| 7-inch tablet screenshots | `tablet-1-menu.png` to `tablet-4-...png` |
| 10-inch tablet screenshots | the same four `tablet-...png` files |

   The tablet screenshots let Google show the game to tablet owners.
4. **App content** (Policy → App content) forms:

| Form | Answer |
|---|---|
| Privacy policy | `https://YOUR-CLOUDFLARE-LINK/privacy.html` |
| App access | All functionality is available without any access restrictions |
| Ads | No, my app does not contain ads |
| Content rating | Start questionnaire → category **Game** → answer **No** to every content question (violence, fear, sex, language, drugs, gambling, user interaction, sharing location, purchases). Expect **Everyone / PEGI 3**. |
| Target audience | Tick **Ages 5 & Under**, **Ages 6–8** and **Ages 9–12** (or just the ones you want). This puts the game under Google's Families rules. |
| Appeal to children | It will ask follow-up questions; answer that the game is designed for children. |
| Data safety | "Does your app collect or share any of the required user data types?" → **No**, then **Save**. |
| Government, financial, health, news | No |

### 1D. The 14-day closed test (required for new personal accounts)

1. **Find 12 testers.** Each needs a Gmail account and an Android phone or tablet. The same 12 people can test all four of your games, but the rule counts for each game separately, so start them in the same week.
2. **Testing → Closed testing → Create track** (or use "Closed testing - Alpha").
3. **Testers:** create an email list with the 12+ Gmail addresses.
4. **Create new release**, upload `NumberPop.aab`, accept **Play App Signing** when asked, write "First test", and roll it out.
5. Copy the **opt-in link** from the Testers tab and send it to everyone. Each tester opens it, taps **Become a tester**, then installs Number Pop Kids from Google Play.
6. All 12 must **stay opted in for 14 days in a row**. Ask them to play a few times. Recruit a couple of extras in case someone drops out.
7. On day 14 or later: **Dashboard → Apply for production**, and answer Google's questions about the test (who tested, what feedback you got, what you changed).
8. Google reviews the application, usually within about a week.

### 1E. Go live

1. **Production → Create new release.** Choose the same `.aab` from the library, add release notes ("First release"), choose countries.
2. **Send for review.** A first release usually takes a few days; kids' games can take a little longer because of the Families review.

---

## Part 2. Apple App Store (iPhone and iPad)

### 2A. Open the account (once, shared by all your games)

1. Make sure your Apple Account has **two-factor authentication** on and uses your **legal name exactly as on your ID**.
2. Install the **Apple Developer** app on an iPhone or iPad. Identity verification happens in that app.
3. In the app: **Account → Enroll Now** → choose **Individual**.
4. Enter your legal name, address and phone, then scan your driver's license or other government ID.
5. Accept the agreement and pay **$99 per year**.
6. Wait for the approval email. Your legal name becomes the seller name on the App Store.
7. Sign in at **appstoreconnect.apple.com** and accept the agreements shown on the home page (the free apps agreement is enough for a free game).

### 2B. Register the game with Apple

1. **developer.apple.com → Account → Certificates, IDs & Profiles → Identifiers → +**.
2. **App IDs → App**. Description `Number Pop Kids`. Choose **Explicit** and enter the app ID from Part 0 (for example `com.numberpop.game`). No extra capabilities. **Register**.
3. **App Store Connect → Apps → + → New App**:
   - Platform: iOS
   - Name: `Number Pop Kids` (if Apple says the name is taken, try `Number Pop Kids: Math Bubbles`)
   - Primary language: English (U.S.)
   - Bundle ID: the one you just registered
   - SKU: `numberpopkids` (any private label)
   - User access: Full access

### 2C. Connect GitHub to your Apple account

GitHub's Mac machines build and sign the iPhone/iPad app, so you don't need a Mac. They need four secrets. If you already made an App Store Connect API key for Word Pop: Definition, reuse the same values and skip to step 7.

1. **Team ID:** developer.apple.com → **Account → Membership details**. Copy the 10-character **Team ID**.
2. **API key:** App Store Connect → **Users and Access → Integrations → App Store Connect API**. If you see **Request Access**, tap it and accept.
3. Under **Team Keys**, tap **+**, name it `GitHub`, access **Admin**, **Generate**. (Admin lets the build create Apple's signing certificate for you.)
4. Copy the **Issuer ID** (above the list) and the **Key ID** (in the key's row).
5. **Download API Key.** You get `AuthKey_XXXXXXXXXX.p8`. Apple lets you download it **only once**: save it somewhere safe and private.
6. Open the `.p8` file in a text editor and copy all of it, including the `BEGIN` and `END` lines.
7. In the `number-pop` GitHub repository: **Settings → Secrets and variables → Actions → New repository secret**, and add:

| Secret name | Value |
|---|---|
| `APPLE_TEAM_ID` | the Team ID |
| `ASC_ISSUER_ID` | the Issuer ID |
| `ASC_KEY_ID` | the Key ID |
| `ASC_KEY_P8` | the whole contents of the `.p8` file |

### 2D. Build and test

1. GitHub → **Actions → Build iPhone and iPad app → Run workflow**. It takes about 10 to 20 minutes.
2. A green check means the game was sent to App Store Connect. Apple then processes it for another 15 to 30 minutes.
3. App Store Connect → Number Pop Kids → **TestFlight**. Under **Internal Testing**, create a group, add yourself, and select the build.
4. Install the **TestFlight** app on your iPhone and/or iPad, open the invite, and install Number Pop Kids. Play every mode, turn the iPad sideways, and check that problems and answers are read aloud.

Each run gives the build a higher number automatically.

### 2E. Store listing

App Store Connect → Number Pop Kids → the **1.0 Prepare for Submission** page:

1. **Screenshots** from the `apple` folder in **number-pop-store-graphics.zip**:
   - **iPhone 6.9" Display:** `iphone-1-menu.png` to `iphone-4-subtraction-correct.png`
   - **iPad 13" Display:** `ipad-1-menu.png` to `ipad-4-subtraction-correct.png` (required, because the game runs on iPad)
2. **Text:** paste the App Store text from "Store text" below.
3. **Support URL:** your Cloudflare link. **Privacy Policy URL:** the privacy page link (under **App Privacy**).
4. **Build:** tap **+** and select the build that came through TestFlight.
5. **App Review Information:** your name, phone and email. No sign-in is needed, so leave the demo account empty. In Notes write: *"Math game for children. No ads, purchases, accounts, links or data collection."*
6. **Age Rating:** answer **None / No** to every content question. It should come out **4+**.
7. **Kids category:** because the name has "Kids" in it, choose **Made for Kids** here. Apple reserves wording like "for kids" in a game's listing for games in the Kids category, so this avoids a rejection over the name. At the end of the age-rating questions, under **Age Categories and Override**, choose **Made for Kids** and the age band **6–8** (or **5 and Under**). This lists Number Pop Kids in the App Store's Kids section. Once in, every future update must keep following the Kids rules (no ads, no tracking, no links out of the game without a parental gate). Number Pop Kids already does. If you choose **Not Applicable** instead, the game is still rated 4+, but Apple's reviewer may ask you to take "Kids" out of the name.
8. **App Privacy:** **Get Started** → "Do you or your third-party partners collect data from this app?" → **No** → **Publish**.
9. **Pricing and Availability:** Free, choose countries.
10. **App Information → Category:** primary **Education**, secondary **Games** (subcategory Educational or Family).

### 2F. Submit

1. Tap **Add for Review**, then **Submit to App Review**.
2. Apple usually replies within a day or two. If they reject it, the message says why: send it to me and I'll fix the issue.

---

## Store text

### Google Play

**App name:** `Number Pop Kids`

**Short description** (80 max):

    Pop the bubble with the right answer! Calm, colorful math for kids.

### App Store

**Name:** `Number Pop Kids`

**Subtitle** (30 max): `Pop the right answer`

**Keywords** (100 max, commas, no spaces):

    math,addition,subtraction,multiplication,division,counting,numbers,bubble,learn,school,calm,autism

**Promotional text** (optional): `Addition, subtraction, multiplication and division with color-changing bubbles. No timer, no ads.`

### Full description (both stores)

    Number Pop Kids turns math practice into calm, colorful bubble popping. A problem appears at the top of the screen, bubbles drift up carrying numbers, and you pop the one with the right answer.

    FOUR KINDS OF MATH, PLUS A MIX
    • Addition
    • Subtraction
    • Multiplication
    • Division
    • Mix them all

    SMALL OR BIG NUMBERS
    Small numbers keeps addition and subtraction up to 10 and multiplication and division up to 5 × 5. Big numbers goes up to 20 and 10 × 10.

    COUNTING PICTURES
    Each problem can show pictures to count: two groups for addition, crossed-out items for subtraction, rows for multiplication and equal groups for division. Turn them off when your child is ready.

    HEARS EVERY PROBLEM
    Every problem and answer is read aloud ("3 plus 4 equals 7!"), and the Count to 100 button counts out loud.

    MADE FOR CALM PLAY
    Color-changing bubbles, no timer and no penalty for a wrong tap, just a gentle wobble. Calm mode slows the bubbles down. Made with kids who stim in mind.

    SAFE FOR KIDS
    No ads, no purchases, no accounts, no links out of the game and no data collected. Works offline. Plays on phones and tablets, including iPad.

---

## Files

| File | For |
|---|---|
| `number-pop.zip` | The game, both build files, this guide |
| `number-pop-play-store-signing-key.txt`, `number-pop-upload-keystore.jks` | Number Pop Kids's Google signing key. Private: never in GitHub. |
| `number-pop-store-graphics.zip` | `google/`: Play icon, feature graphic, phone and tablet screenshots. `apple/`: iPhone (1290 × 2796) and iPad (2064 × 2752) screenshots. Kept out of the game zip so it stays small enough to upload to GitHub. |
| `assets/icon-1024.png` | The App Store icon (the build adds it automatically) |
| `AuthKey_XXXXXXXXXX.p8` | Apple API key you download in 2C. Private: never in GitHub. |

## If a build fails

**Actions** → the failed run → the job → the step with the red ❌. Send me the step's name and the red lines near the bottom. The iPhone build's signing step is the one most likely to need adjusting on its first run.

## Costs

| | Google Play | App Store |
|---|---|---|
| Account | $25 once | $99 per year |
| Per game | Free | Free |
| GitHub builds | Free for public repositories | Free for public repositories |
