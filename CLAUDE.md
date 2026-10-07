# CLAUDE.md — Drik Panchangam

Telugu Panchangam app with real astronomical (Drik Ganitham) calculations.
Read `HANDOFF.md` for features, validation history and Capacitor steps.

## Architecture
- The whole app is ONE file: `www/index.html` (HTML + CSS + JS, ~3,350 lines). No build step, no external libraries, no backend. Keep it that way unless the owner asks otherwise.
- `source.html` is an old backup copy. Do not edit it; `www/index.html` is the source of truth.
- Capacitor wraps `www/` for Android/iOS. After editing, run `npx cap sync`.
- Test the web version by opening `www/index.html` in a browser (or `npx serve www`).

## Hard rules
1. **Never ship unverified astrological data.** The owner's standing instruction: no guessed or unreliable content. Adaya-Vyaya and Rajapujya-Avamana are deliberately NOT implemented (no verifiable formula found). Do not add them unless a real classical source with the actual computation is provided and checked against published data.
2. **Every user-visible string must support 4 languages**: English (en), Telugu (te), Hindi (hi), Sanskrit (sa).
   - Static HTML: `data-en`, `data-te`, `data-hi`, `data-sa` attributes, applied by `applyLang()`.
   - Dynamic JS text: `pick(L,en,te,hi,sa)` and `pickArr(L,idx,EN,TE,HI,SA)`. Falls back to English if missing, so a missing translation is silent. Check Telugu/Hindi/Sanskrit modes explicitly after any UI change (alerts, placeholders, empty states included).
3. **Sunrise anchoring**: all primary panchangam values (tithi, nakshatra, yoga, karana, masa, ayanam) are computed at the local sunrise JD in `computeAll()`. Only `nowTithi` ("Right now" chip) uses the current clock. Do not change this convention.
4. **Tithi display** is `start(-) – end(+)`: "(-)" = start on the previous day, "(+)" = end on the next day. Found via `findCrossing()`.
5. **Storage** is local only, through the `appStorage` shim over `localStorage`. Keys: `life-events`, `lang-pref`, `default-location`. No login, no sync, no network calls.
6. Indexing: masa 0=Chaitra; weekday 0=Sunday; nakshatra 0=Ashwini.

## Astronomy core (do not "simplify")
- Lahiri ayanamsa: 23.856° at J2000 + 50.2388475″/yr.
- Sun/Moon: Meeus-style low-precision series. Mercury–Saturn: orbital elements. Rahu/Ketu: mean node.
- Masa: amanta, lunation index (`NEWMOON_REF_JD`, `SYNODIC`), with Adhika detection.
- Mandi = ascendant at start of Gulika Kalam.

## Known validated results (use as regression checks)
- Jan 7, 2026 (kshaya case): tithi 19, start 8:03 AM(-), end 6:53 AM, sunrise 6:48 AM.
- Ugadi Mar 19, 2026 = Parabhava, Shaka 1948.
- Navanayakulu formula matched the published 2022-23 example 9/9.
- Muhurtham: Feb 6, 2026 found for Gruhapravesam; Chaturmas excluded for Vivaham.
- Rahu ~306.70° (reference 306.67°). Planets within 0.5–2.5° of a July 2026 ephemeris.

## Open items
- Verify Panchanga Sravanam role assignments: `findCrossing(sunSidAtJD,targetDeg,ugadiJD+30,200)` may pick the previous year's crossing for some roles. Check against a published Panchangam.
- Recheck Telugu mode empty states (e.g. "No birthdays or anniversaries added yet.") for untranslated English.
- No automated tests exist. Extract the astronomy functions and add a regression test suite from the results above.
- No icons/splash/privacy policy. Placeholder appId `com.drikpanchangam.app`.
- Native geolocation: switch to `@capacitor/geolocation` and add permission strings.

## Working style
- Make small, targeted edits. Explain what changed and how you verified it.
- Do not reformat or reorder the file wholesale.

## Local project state (added when moved to Claude Code)
- This Mac already has `node_modules/` and an `ios/` Capacitor project (`npx cap add ios` was run). Android (`npx cap add android`) has not been added yet.
- `www/index.html` here may differ from the chat-session copy. Treat the local `www/index.html` as the source of truth; do not overwrite it.
- Always run `npx cap sync` after editing `www/index.html`.
