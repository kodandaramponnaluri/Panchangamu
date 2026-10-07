# Drik Panchangam — Developer Handoff

**Status as of this document (updated 2026-08-17):** Working single-page app, validated extensively, wrapped for iOS via Capacitor and confirmed running in the iOS Simulator (iPhone 17 Pro, iOS 26.5). Panchangamu and Jathakamu tabs verified visually — data renders correctly, geolocation permission prompt works. Android not yet attempted (no Android Studio on this machine yet). Project now lives at `~/Developer/panchangam-app` on this Mac.

### What was done in this session
- `npm install` succeeded (Capacitor 6.x core/cli/ios/android + geolocation plugin).
- `npx cap add ios` scaffolded `ios/App`; `pod install` initially failed on a CocoaPods/Ruby UTF-8 locale error — fixed by exporting `LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8` before running `pod install` / `cap sync`. If this bites again, that locale export is the fix.
- Added `NSLocationWhenInUseUsageDescription` to `ios/App/App/Info.plist` (was missing — required before first build, per Section 5 below).
- Built via `xcodebuild` for the simulator and ran on iPhone 17 Pro: app launches, location permission prompt appears and works, city picker works, Panchangamu tab shows correct live data (tithi, nakshatram, yoga, karana, samvatsara, moon phase) for Hyderabad on Aug 17, 2026, Jathakamu birth-details form renders correctly.
- Note: `xcrun simctl privacy grant location <bundle-id>` did NOT reliably suppress the system prompt in this session (kept reappearing after grant+relaunch) — just tap through the permission dialog manually instead of relying on pre-granting.

### Still outstanding from Section 6 below
- Android: no Android Studio installed on this machine yet — `npx cap add android` was not run.
- App icons/splash screens, privacy policy/ToS, further Muhurtham event types, Adaya-Vyaya (still deliberately unimplemented).

---

## 1. What this app is

A Telugu Panchangam (Hindu calendar/almanac) app with real Drik Ganitham (position-based) astronomical calculations — not a lookup table. Built entirely as one self-contained HTML file (`www/index.html`): no build step, no external libraries, no backend. Everything — astronomy, UI, translations — lives in that one file.

### Tabs / features
| Tab | What it does |
|---|---|
| **Panchangamu** | Daily panchangam: Tithi, Nakshatram, Yoga, Karana, Masam, Samvatsara, Ayanam, sunrise/sunset, Rahukalam/Yamagandam/Gulika Kalam, Varjyam, Durmuhurtam. Personal birthdays/anniversaries list with lunar-date matching. |
| **Jathakamu** | Birth details → Masam/Tithi/Nakshatram/Rashi, a South Indian-style Jathaka Chakram (birth chart) with all 9 grahas + Mandi + Lagna, and a Vimshottari Dasha timeline (next 10 years). |
| **Event Lookup** | Forward (date → panchangam) and reverse (Masam+Tithi → next Gregorian date) lookup, for birthdays/anniversaries by lunar date. |
| **Telugu Calendar** | Month grid with moon-phase icons, Masam/Ayanam/Samvatsara transition markers, kshaya-tithi handling (chains through tithis a naive sunrise-only display would skip). |
| **Festivals** | Current + next year's festival dates, validated against real published sources. |
| **Kundali Match** | Ashtakoot Guna Milan (8-factor, 36-point) marriage compatibility between two people. |
| **Muhurtham Finder** | Auspicious-date search for Gruhapravesam, Vivaham, Namakaranam, Aksharabhyasam — scores candidate days against published rule sets. |
| **Panchanga Sravanam** | Ugadi year-forecast: Samvatsara overview, Navanayakulu (nine planetary year-rulers), Desha Phala, Rasi Phalalu with associated birth-star (Nakshatra) listings down to the exact pada. |

All of the above work in **English, Telugu, Hindi, and Sanskrit** (toggle in the header and on first-launch welcome screen), and are **location-aware** (GPS or manual city picker).

---

## 2. What's been validated, and how

This matters for anyone picking up the code — the astronomy isn't guesswork, and each piece was checked against something external before shipping:

- **Sun/Moon/Ayanamsa**: standard low-precision series (Meeus-style), Lahiri ayanamsa anchored correctly at J2000 (an early bug had it anchored at 1950 — fixed).
- **Masa/Adhika-masa detection**: lunation-index approach, validated against real 2026 Adhika Jyeshtha dates.
- **Navagraha positions (Mercury–Saturn)**: paulschlyter-style orbital elements. Checked against a published July 2026 ephemeris — all 5 planets landed within 0.5–2.5° of reference, which is expected/sufficient accuracy for correct rashi placement.
- **Rahu/Ketu**: mean node formula, matched a published position almost exactly (306.70° vs. reference 306.67°).
- **Lagna (Ascendant)**: validated two ways — against a general formula, and via an independent physical sanity check (Ascendant should equal the Sun's longitude at the moment of sunrise; came out within 0.86°).
- **Mandi**: computed as the Ascendant at the start of that day's Gulika Kalam period.
- **Vimshottari Dasha**: internal consistency checks (antardashas sum exactly to their mahadasha's duration; all 9 mahadashas sum to exactly 120 years) plus matched a published worked example (Moon at 14°27'15" Cancer → Saturn dasha balance ≈3.153 years) to the 4th decimal.
- **Festival dates**: cross-checked against multiple real published 2026 calendars, including catching and fixing a real bug (Ugadi was silently missing some years because the sunrise-tithi rule fails on short/kshaya Pratipada — switched to a noon rule).
- **Kshaya-tithi handling**: the Telugu Calendar correctly detects and displays tithis that would otherwise be skipped entirely by a naive sunrise-only day label — verified against a real 2026 case (Jan 7, 2026).
- **Ashtakoot Kundali matching**: all 8 kootas implemented from standard published tables; scoring tested against synthetic self-match, mismatched-pair, and known-good-pair cases for sanity.
- **Navanayakulu (Panchanga Sravanam)**: formula validated to a **9/9 exact match** against a real published 2022-23 (Shubhakruth) example.
- **Muhurtham finder**: validated against real published 2026 muhurtham lists — correctly excludes the Chaturmas window entirely for marriage dates, and independently found Feb 6, 2026 for Gruhapravesam, matching a published reference exactly.

### Explicitly NOT implemented (by design, not oversight)
- **Adaya-Vyaya / Rajapujya-Avamana** numeric ratios (income/social-standing scores per rashi, shown in some published Panchangams). Investigated at length — found real published data for 2 consecutive years and confirmed structural facts (Adaya-Vyaya is planet-based, not rashi-based), but could not pin down a verifiable public formula for the exact year-to-year values, and two "formulas" surfaced from further research both failed against real data when tested. Deliberately left out with an honest in-app note rather than shipping a guess. If a real classical-text source ever turns up, revisit this.

---

## 3. Known technical debt / gaps for whoever continues this

- **Jathakamu's horoscope-glimpse trait text**: fully translated now (was previously falling back to English for Hindi/Sanskrit — fixed).
- **Storage**: just converted from `window.storage` (a Claude-artifact-only API) to plain `localStorage` via a small shim (`appStorage` in the code) — see Section 5. This was necessary for porting; nothing else changed in the calling code.
- No automated tests exist. All validation so far has been manual/scripted one-off checks (see Section 2) — worth writing a real test suite if this becomes a maintained product, especially around the astronomy core and the Vimshottari Dasha date math.
- No app icons, splash screens, or store-listing assets exist yet.
- No privacy policy / terms of service (required for App Store / Play Store submission).

---

## 4. Folder structure

```
panchangam-app/
├── HANDOFF.md              ← this file
├── package.json            ← Capacitor project config (not yet npm-installed)
├── capacitor.config.json   ← Capacitor app config (appId, appName, webDir)
├── source.html             ← original single-file app (kept as reference/backup)
└── www/
    └── index.html          ← the actual app, as Capacitor expects it (webDir target)
```

After running the setup steps below, `ios/` and `android/` native project folders will appear alongside `www/`.

---

## 5. Setting this up as a real Capacitor project

This hasn't been run yet — these are the exact next steps.

### Prerequisites
- Node.js and npm installed
- For Android: Android Studio (works fine on Windows)
- For iOS: **a Mac is required to build the final binary** — Xcode doesn't run on Windows or Linux. Given the current dev machine is Windows, the practical options are:
  1. A cloud Mac rental (MacStadium, MacinCloud) for occasional builds
  2. A CI service with macOS runners that supports Capacitor (Codemagic and Bitrise both have Capacitor-specific docs; GitHub Actions also offers macOS runners you can script with `fastlane`)
  3. Borrowing/buying Mac access when ready to actually ship to the App Store

  None of this blocks *development* — you can build and test the Android version entirely on Windows, and test the web version in any browser, while the iOS path is figured out separately.

### Steps
```bash
cd panchangam-app
npm install

# Add native platforms (creates ios/ and android/ folders)
npx cap add android
npx cap add ios      # only works for the final Xcode step; the CLI itself runs fine cross-platform

# Whenever www/index.html changes, sync it into both native projects
npx cap sync

# Open in the native IDE
npx cap open android    # opens Android Studio
npx cap open ios        # opens Xcode (Mac only)
```

### Before the first real build, also do:
1. **Geolocation permissions** — the app calls `navigator.geolocation.getCurrentPosition()` directly. This generally works inside a Capacitor WebView, but for reliable native permission prompts, consider swapping to the official `@capacitor/geolocation` plugin (already listed in `package.json`) and adding the required permission strings:
   - iOS: `NSLocationWhenInUseUsageDescription` in `ios/App/App/Info.plist`
   - Android: location permissions in `android/app/src/main/AndroidManifest.xml` (Capacitor's Android geolocation setup docs cover the exact lines)
2. **Print button** (birthday list export) — `window.print()` may not behave the same in a native WebView on all platforms. Test it; if it doesn't work well, the Share/Email/Copy buttons already provide working alternatives, or swap to `@capacitor/share`.
3. **App icons & splash screens** — Capacitor has an `@capacitor/assets` tool that generates all required sizes from one source image; nothing exists yet to feed it.
4. **appId** — currently set to `com.drikpanchangam.app` as a placeholder in `capacitor.config.json`. Change this to whatever reverse-domain identifier you actually want to publish under (can't be changed later without a new listing).

### Web deployment
The `www/index.html` file is also just a normal static website — deployable to Netlify/GitHub Pages/etc. independently of the mobile work, exactly as discussed earlier. No changes needed for that path; it already works standalone.

---

## 6. Where to pick this up

Suggested order:
1. `npm install` + `npx cap add android` + `npx cap sync` + open in Android Studio, confirm it runs in an emulator.
2. Fix geolocation permissions for Android specifically first (faster iteration loop than iOS).
3. Get a Mac/cloud-Mac path sorted for iOS in parallel, since that lead time is longer.
4. Generate app icons/splash screens.
5. Decide on the two remaining backlog items from earlier in this project: further Muhurtham event types, and (if a real source ever surfaces) Adaya-Vyaya/Rajapujya-Avamana.
