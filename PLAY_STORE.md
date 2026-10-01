# DAFHIO — Google Play release checklist

## Compliance check (Play Console steps)

| Step | Requirement | Status |
|---|---|---|
| 2 | Android App Bundle (.aab) | ✅ `npm run android:bundle` → `dist/DAFHIO Education Game (Demo).aab` (built from the `play` flavour) |
| 2 | Signed with a release keystore | ✅ build.gradle reads `android/keystore.properties` — **you create the keystore** (see below) |
| 2 | Target API 35 (Android 15) | ✅ `targetSdkVersion = 35`, `compileSdkVersion = 35` (was 34). Needs Android SDK Platform 35 installed in Android Studio. Recommended later: upgrade to Capacitor 7 (`npm i @capacitor/core@7 @capacitor/android@7 @capacitor/cli@7`, minSdk becomes 23) |
| 2 | Android 15 edge-to-edge | ✅ `viewport-fit=cover` + safe-area padding, so the HUD stays clear of the status / navigation bars |
| 4 | Privacy policy URL | ⚠ page ready (`pages/privasi.html`) — **publish it online** (e.g. GitHub Pages) and paste the URL |
| 4 | App access | ✅ no login, everything open → declare "All functionality available without special access" |
| 4 | Ads | ✅ no ads, ever → "No" |
| 4 | In-app purchases | ✅ none yet (the demo shows a disabled "Beli — Akan datang" button, no payment code). Declare IAP only when billing is added |
| 4 | Content rating | ⚠ fill the questionnaire: education game, cartoon mild peril (dogs, tree monster, ghost), no violence/gambling, user interaction only on local LAN → expect Everyone / PEGI 7 |
| 4 | Target audience | ⚠ select 6–8 and 9–12 → Families policy applies (no ads SDK, no data collection — already true) |
| 4 | Data safety | ✅ "No data collected, no data shared" (saves stay on the device; LAN is device-to-device) |
| 5 | Name ≤ 30 chars | ✅ "DAFHIO Education Game" (21) |
| 5 | Short description ≤ 80 | ✅ see below (BM 55 / EN 64 chars) |
| 5 | Full description ≤ 4000 | ✅ template below |
| 5 | Icon 512×512 | ✅ `assets/icon-512.png` |
| 5 | Feature graphic 1024×500 | ⚠ to make (can be rendered from `scripts/icon/icon.html` at 1024×500) |
| 5 | 2–8 screenshots | ⚠ take with photo mode (P) or the emulator |
| 6 | Closed test 14 days (new personal accounts) | ⚠ needs 12+ testers opted in for 14 days before production |
| 7 | Review | after the above, "Send for review" |

**Create the keystore once** (keep it forever, back it up):
```
keytool -genkey -v -keystore dafhio-release.jks -alias dafhio -keyalg RSA -keysize 2048 -validity 10000
```
then `android/keystore.properties`:
```
storeFile=../../dafhio-release.jks
storePassword=…
keyAlias=dafhio
keyPassword=…
```

## Build

1. `npm run build:web` then `npm run android:sync`.
2. **Google Play upload:** `npm run android:bundle` → `dist/DAFHIO Education Game (Demo).aab`.
   Test APK of the same app: `npm run android:release` → `dist/DAFHIO Education Game (Demo).apk`.
3. **Private Full APK** (family, sideload only): `npm run android:full` →
   `dist/DAFHIO Education Game (Full).apk`.
4. Bump `versionCode` every upload (`-PbbVersionCode=14 -PbbVersionName=1.1`);
   the build scripts already pass these.
5. Sign with your **upload key**. Keep the keystore file + passwords in two safe
   places (you cannot update the app without it). Turn on *Play App Signing*.

Current project: minSdk 22 (Android 5.1), targetSdk 35, package
`my.dafhio.game`. Permission: INTERNET only (LAN play). Camera is asked
only when scanning a friend's QR code.

## Store listing

- **App name:** DAFHIO Education Game
- **Short description (BM):** Dunia sekolah 3D: belajar, kuiz, sukan & kawan-kawan!
- **Short description (EN):** A 3D Malaysian school adventure: learn, quiz, play & make friends!
- **Full description:** school life RPG for Darjah 1–6 — Matematik, Sains, BM,
  English quizzes from the KSSR-style bank; taman, waterpark, pasar malam,
  library, sports complex, surau, Rumah Budaya, arcade; friends & LAN play.
- **Icon:** `assets/icon-512.png` (512×512). **Feature graphic:** 1024×500
  (make from `scripts/icon/icon.html` with a wider canvas).
- **Screenshots:** at least 2 (recommend 5) phone screenshots, 16:9 landscape
  — use photo mode (P) in the game, or the emulator.
- **Category:** Education (or Game → Educational). **Tags:** Education, Kids.

## Policy

- **Privacy policy URL** (required): publish `pages/privasi.html` (e.g. GitHub
  Pages) and paste the URL. The same page is linked in Tetapan.
- **Target audience:** 6–12 → the app is in the *Designed for Families* /
  Teacher Approved rules: no ads, no personal data collection, no external
  links without a parent gate.
- **Content rating questionnaire:** Education · cartoon mild peril (dogs, tree
  monster, ghost) → expected *PEGI 7 / Everyone*.
- **Data safety form:** "No data collected", "No data shared" (LAN play is
  device-to-device on the local network, chosen by the user).

## Test before release

- Two Android phones minimum (one mid-range, e.g. Redmi Note / Samsung A).
- Tetapan → Grafik: Auto should pick *Sederhana* (or *Rendah* on ≤ 3 GB RAM).
- Battery below 20 % → the game drops to 30 fps automatically.
- Try LAN play on a hotspot: host on PC, join with the room code.

## iOS (later)

Same web build wrapped with Capacitor iOS (`npx cap add ios`), Xcode for
signing, bundle id `my.dafhio.game`, Local Network usage description for
LAN play, TestFlight for testers.

## Editions: free demo now, full version later (one app)

The Play app is **one app** (`my.dafhio.game`). It ships as the free demo;
the full version will be unlocked **inside the same app** with a one-time
Google Play in-app purchase, so players never reinstall and keep their saves.

- Switch: `D.config.EDITION` (`js/game/data/gameConfig.js`) is `'demo'` unless
  `window.DAFHIO_EDITION` (`js/edition-flag.js`) says `'full'`.
- Run-time unlocks: `D.edition.unlock('full_version')` (and later packs, e.g.
  `'pack_bandar'`), stored on the device in localStorage `bb_edition_unlocks`
  so every student profile gets them. Code: `js/game/gameplay/Edition.js`.
- Demo limits (`D.config.DEMO`): Taman Air, Pasar Malam, Paintball and Pendekar 1 free visit per
  game week each · UASA 1 trial paper · ranks above Undergraduate shown as
  "Versi Penuh" (score keeps counting, pocket money capped at RM15) · Kereta
  and the 🏁 Litar Lumba (car racing + garage) locked. Learning quizzes are
  never locked. No ads, no nagging pop-ups.

**Store listing wording (honest):** "Versi percuma dengan kandungan terhad.
Versi Penuh (Taman Air & Pasar Malam tanpa had, semua kertas UASA, pangkat
hingga Guru Besar, kereta & litar lumba) akan datang sebagai pembelian dalam
aplikasi." Do not promise dates.

### When you are ready to sell the full version

1. Play Console → **Monetise → Products → In-app products** → create
   `full_version` (one-time, e.g. RM9.90). Later packs: `pack_bandar`, …
2. Add a billing plugin to the Android build — `cordova-plugin-purchase`
   (CdvPurchase) or RevenueCat (`@revenuecat/purchases-capacitor`).
3. Wire it: on a successful or restored purchase of `full_version` call
   `D.edition.unlock('full_version')`; enable the "Beli" button in the Versi
   Penuh panel to start the purchase; run "restore purchases" at start so a
   reinstall or a new phone unlocks again.
4. Update the Data safety form (purchase history handled by Google Play) and
   declare in-app purchases on the store listing.

### Free full version for your own children

- **Promo codes** (best): *you* create them in Play Console → **Monetise →
  Promo codes → Create promotion** for `full_version`; Google generates the
  codes (about 500 per quarter). Redeem in Play Store → *Redeem code* or on the
  purchase screen. It is a real, permanent free purchase on that Google
  account, restored by the normal purchase restore.
- **License testers** (testing only): Play Console → **Settings → License
  testing** → add Gmail accounts; they buy with a test card, no charge.
- **Full APK** (`npm run android:full`): app id `my.dafhio.game.full`, name
  "DAFHIO Education Game (Full)", everything open. Sideload only: it does not
  update from Play and has its **own separate saves** (use 💾 Sandaran /
  📂 Pulihkan on the profile screen to move a profile across).

Gradle flavours (`android/app/build.gradle`): `play` (the store app) and
`full` (`applicationIdSuffix '.full'`, `src/full/res/values/strings.xml`,
`src/full/assets/public/js/edition-flag.js`). Keystore: alias `dafhio`, file
`dafhio-release.jks` — the same key signs both.

## Build tools on this PC (tested 29 Sep 2026)

- Android SDK: `%LOCALAPPDATA%\Programs\android-build\android-sdk` (in `android/local.properties`),
  platform **android-35** and build-tools 35.0.0 installed.
- Java: the JDK 17 in `%LOCALAPPDATA%\Programs\android-build\jdk` (the Java 8 on PATH is too old).
- `npm run android:*` call `node scripts/gradle.js`, which picks that JDK and works round the
  Windows JDK 17 error "Unable to establish loopback connection" by pointing
  `jdk.net.unixdomain.tmpdir` at `C:\udtmp`.
- Result of the test build: Play APK (`my.dafhio.game`, "DAFHIO Education Game", edition demo),
  Full APK (`my.dafhio.game.full`, "DAFHIO Education Game (Full)", edition full), Play AAB —
  all target SDK 35. Unsigned until `android/keystore.properties` exists.
