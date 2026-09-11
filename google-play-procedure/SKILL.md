---
name: shipping-expo-apps-to-google-play
description: Use when taking a finished Expo/React Native codebase to Google Play for the first time, setting up Android Google Sign-In, running an EAS cloud build, or working through Play Console submission and the 12-testers-14-days closed testing requirement
---

# Shipping an Expo app to Google Play

## Overview

The procedure, in forced order, for a codebase that is already finished.

**One principle governs everything: native config is frozen at build time, JavaScript is not.**
Anything native you forget costs a new build, a new upload and **another review**. Anything in
JS ships over the air for free, the same day. So the whole game is enumerating the native half
BEFORE the first build, and not agonising over the rest.

**A green build is not a correct build.** Verify by unpacking the artifact. Four separate
defects in one real submission produced builds that compiled, uploaded and ran while being
wrong — none of them findable by search, because none of them errored.

## Phase 0 — the freeze list (do this before `eas build`)

Everything here is compiled into the binary. Missing one = a full extra review cycle.

- [ ] **Every native module and config plugin present**, even if the feature ships disabled.
      Crash reporting, notifications, maps, auth SDKs. Adding one later is a new native build.
- [ ] **`google-services.json` committed and wired** (`android.googleServicesFile`) if push may
      EVER be wanted. It is compiled into Android resources by the Gradle plugin and **cannot
      arrive over the air** — there is no runtime API to point FCM at a project. With the file
      in, enabling push later is a server variable. Without it, a v1.1 store release.
- [ ] **Permissions reconciled with what you will declare.** Block unused ones in app config.
      A manifest that declares location while Data safety says otherwise is a rejection.
- [ ] **Icons final.** Native, so every change is a new build.
- [ ] **All `EXPO_PUBLIC_*` values in `eas.json`'s `build.<profile>.env`, not `.env`.**
      `.env` is gitignored and **EAS cloud builds never read it**.
- [ ] **`eas.json` contains no comment keys.** The schema is strict; JSON has no comments.
      `"_note": "..."` makes the build refuse to start.
- [ ] **Monorepo: workspace packages build on the builder.** If a package points `main` at a
      gitignored `dist/`, add `"eas-build-post-install": "<build command>"` to the app's
      package.json. Local scripts usually build them first, which hides this until the cloud.
- [ ] **`version` set deliberately.** OTA updates only reach builds with a matching version.

**Then verify the config resolves before spending a build:**
```bash
npx expo config --type public --json | head -40
```

## Phase 1 — build

```bash
npx eas-cli@latest login          # package is eas-cli; `npx eas` fails
npx eas-cli@latest build --profile production --platform android
```

Runs on Expo's servers, not locally. Free tier ≈ 15 builds/month. **Do not run `eas init` on an
already-linked project** — it prompts, and a wrong answer mints a second project id and a second
update channel.

The first build generates the **upload keystore**, which signs every future update. Back it up;
EAS holds the only copy otherwise.

**Verify the artifact, do not trust the tick:**
```bash
unzip -q app.aab -d out
grep -ac "<expected value>" out/base/assets/index.android.bundle   # expect >=1
```
Use **`grep -a`**. A Hermes bundle is binary and plain grep silently reports nothing for a binary
file — a false negative that reads as "it never compiled in".

## Phase 2 — the credentials chain (order is forced)

```
eas build ──→ upload key SHA-1 ──→ Android OAuth client
     │
AAB upload ──→ Play App Signing SHA-1s ──→ same OAuth client   ← BEFORE any tester installs
     │
  roll out ──→ testers opt in ──→ 14 days
```

Each step creates the input the next needs. You cannot reorder them.

**Google Sign-In, the three things people get wrong:**

1. **The app configures the SDK with the WEB client id**, never the Android one. The Android
   client exists only so Google can match `(package name, signing certificate)`; it is never
   referenced in code. The web client id is the token's audience, which your backend validates.
2. **One SHA-1 per OAuth client.** No multi-fingerprint field exists. Create **one client per
   certificate, all sharing the package name** — four is normal. The app is unchanged.
3. **Both OAuth clients must be in the same GCP project.** Compare the project-number prefix of
   the two client ids. A Firebase project being a different GCP project is fine and expected.

**Register every certificate, especially Play's.** Google re-signs your upload with its own key,
so store installs run a different certificate than anything you build. Registering only the
upload key gives you sign-in that works perfectly on your device and is **dead for every real
user** — the most expensive shape of this bug, because nothing you test reveals it. Hybrid
signing shows three Play keys; register all.

**Test the real binary before review:** Play Console → **App bundle explorer → Downloads →
signed universal APK**. That carries Play's signature, so installing it proves the exact chain
users get. Uninstall any directly-built copy first (`INSTALL_FAILED_UPDATE_INCOMPATIBLE` just
means a different signature).

## Phase 3 — console

Console layout and policy change constantly. **Search the current
`support.google.com/googleplay/android-developer` pages on the day** rather than trusting any
summary. Where a summary and the console disagree, the console is right.

What is stable enough to state:

- **Data safety must match the binary.** The merged manifest's permissions are checked against
  it.
- **Content rating, app access, target audience** all gate the release.
- **App access needs a working demo account.** A reviewer will use it. Never share it with testers.
- Review inspects the **app**, not just declarations: static analysis of the binary, a device-farm
  run (results in **Pre-launch report** — read it), and human review for new accounts and UGC.

## Phase 4 — closed testing

Personal accounts created after 2023-11-13 need **12 testers opted in continuously for 14 days**
before applying for production access.

- **Use the `alpha` closed track. Internal testing does NOT start the clock.**
- **The closed track has its own country list**, separate from production. Narrow it and testers
  abroad hit "App not available in your country".
- **Adding tester emails starts nothing.** The clock runs from actual opt-ins via
  `https://play.google.com/apps/testing/<package>`.
- **Enrol 15–16.** Below 12 pauses or resets the counter.
- The developer account is not automatically a tester.

**The application is graded on engagement, not headcount.** It asks what feedback you collected,
how usage compared to expected production behaviour, and **what you changed because of it**. So
during the window: ship ~3 OTA updates (`eas update`, free, no review), collect a sentence from
each tester, watch the count daily. If the app is not in the testers' language, send a numbered
brief of what each button does, or they bounce in ten seconds and the record shows it.

## Irreversible, or expensive to undo

| Decision | Cost of changing |
|---|---|
| **Free vs Paid** | **Permanent.** A free app can never become paid |
| **Package name** | Fixed by the first upload, forever |
| **Upload keystore** | Signs every update. Lost = Play support ticket |
| **Play App Signing enrolment** | Automatic and permanent for new apps |
| **Developer name shown on the listing** | Your KYC-verified legal name |
| **Anything native** | New build + new upload + **another review** |
| **The 14-day clock** | Cannot be restarted without losing the days already served |

## Free to change later — do not block on these

JavaScript, copy, layout, screens, API wiring (`eas update`, same day, no review) · store
listing text, graphics, screenshots · Data safety and rating answers · countries · server-side
config and feature flags.

**Obsess over the first table. Ship on the second.**

## The four silent failures

| Symptom | Cause |
|---|---|
| Build dies in **Bundle JavaScript**, names no file | monorepo packages never built on the builder |
| `eas.json is not valid — "_x" is not allowed` | comment keys in eas.json |
| Feature **silently absent** from a cloud build | `EXPO_PUBLIC_*` only in gitignored `.env` |
| Sign-in works for you, **dead for store installs** | Play App Signing SHA-1 not registered |

The last two share a shape worth remembering: **a value that can go missing produces a build
that is quietly wrong rather than broken.** Prefer committed config over `.env` for anything a
cloud build needs.
