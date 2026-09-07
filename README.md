# practicemill.com — static files the apps rely on

Everything under `web/` is meant to be served verbatim from `https://practicemill.com/` (apex, HTTPS, no redirect). Nothing in the app reads these files; the platforms' password managers do.

## `.well-known/apple-app-site-association`

Lets iOS Password AutoFill and strong-password generation link the app to the domain once the app carries the Associated Domains entitlement `webcredentials:practicemill.com` — NOT yet in the repo, see *Enabling the iOS side* below. Requirements: served over HTTPS at `/.well-known/apple-app-site-association`, `Content-Type: application/json`, **no** `.json` extension, no redirect. Apple's CDN fetches it when the app is installed; changes take up to a day to propagate.

Content: team id `BRB4J6X2V3` + bundle id `com.dervaux.PracticeMill`.

## `.well-known/assetlinks.json`

Same link for Android's autofill / Google Password Manager (Digital Asset Links). Must be served at `/.well-known/assetlinks.json` with `Content-Type: application/json`.

Fingerprints listed:
- `F8:9F:E5:…:14:49` — the **debug** keystore (`~/.android/debug.keystore` on Félix's Mac). With no `android/key.properties`, `flutter build apk --release` falls back to debug signing, so this is what sideloaded tester APKs carry.
- **To add when the app goes to Google Play:** Play App Signing re-signs uploads; copy the *app signing key* SHA-256 from Play Console → Setup → App signing into the array (keep the debug one while sideloading continues).

Verify after upload: `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://practicemill.com&relation=delegate_permission/common.get_login_creds`

## Enabling the iOS side (one-time, in Xcode)

The `webcredentials:practicemill.com` entitlement is NOT in the repo yet: a provisioning profile without the capability refuses to sign (`doesn't include the Associated Domains capability`), and only an Apple-ID-signed Xcode can register it. Do it once: open `Flutter/ios/Runner.xcworkspace` → target Runner → *Signing & Capabilities* → *+ Capability* → *Associated Domains* → add `webcredentials:practicemill.com`. Xcode registers the capability on the App ID, regenerates the automatic profile, and writes the entry into `Runner.entitlements` — commit that file afterwards.

## Hosting

GoDaddy's parked page cannot serve custom paths. Any free static host with a custom domain works: point the domain's DNS at it from GoDaddy (keep GoDaddy as registrar) — Cloudflare Pages, Netlify, or GitHub Pages (add an empty `.nojekyll` so the dotted folder is served). Upload this `web/` folder as the site root.
