---
name: add-app-clip
description: Add an iOS App Clip for Flow Affiliate invites to an Expo app, or check the one it has. The clip opens from a publisher's link without an install, greets the invited person, offers the full app and hands the click over through an App Group. Use when the user asks about an App Clip for Flow Affiliate, or when integrate-flow-affiliate suggests one.
---

# App Clip for Flow Affiliate

With an App Clip, an iPhone without the app opens a small part of it straight from a publisher's link: it shows who invited them and the bonus, offers the full app, and leaves the click in an App Group. The full app reads it on first launch, so the sign-up is credited with no code to type. It is optional: without it, iPhones go to the App Store and the install is matched to the click on first launch.

## 1. Where things stand

Call `get_setup` and `app_clip_guide` (the `flow-affiliate` MCP server; if the tools are missing, see step 0 of `integrate-flow-affiliate`).

- The program converts only on a website: an App Clip doesn't apply. Say so and stop.
- The program **has an App Clip** (`iphone.route` is `app_clip`): go to step 4 and check it.
- It **has none**: explain in two or three lines when it helps (publishers whose audience is on iPhone; iOS 16.4 or later; some in-app browsers may not show the App Clip card) and ask whether to add one. Continue only on a yes.

## 2. Before you add one

Inspect the project and say what you found:

- An Expo app (`expo` in `package.json`) with an iOS bundle identifier (`ios.bundleIdentifier`). The clip's bundle ID is that plus `.Clip`; the App Group is `group.<bundle id>`.
- The Expo SDK version: `Paths.appleSharedContainers` in `expo-file-system` needs SDK 54 or later. On older SDKs, use a shared-group preferences library behind the same `getItem` / `setItem`.
- That `@flow/expo` is already installed and wired (`integrate-flow-affiliate`). If not, do that first.
- Which native modules the clip can leave out (notifications, camera, analytics, payments, updates).

## 3. Add it

Follow `app_clip_guide` step by step, with the program's real values:

1. `react-native-app-clip` in app.config, with `groupIdentifier`, `deploymentTarget: '17.0'` and `excludedPackages`. `expo-updates` must always be excluded.
2. The same App Group in the full app's entitlements.
3. Metro: with `BUILDING_FOR_APP_CLIP`, prefer `*.clip.*` files, and add `src/entry.clip.tsx` (if the app's entry isn't already a file that `index.js` imports, make it one).
4. `groupStorage` over the App Group.
5. The clip's screen: `useLinkingURL()` for the invocation URL, `saveHandoff({ code: fg_ref, clickId: fg_click, url }, groupStorage)`, `GET /v1/share/landing/:code` with a 2-second timeout for the greeting and the bonus, and "Install the app" with `displayOverlay()`. Keep it to one or two screens, in the app's own look and language.
6. The full app: `handoff: Platform.OS === 'ios' ? groupStorage : undefined` in `createAffiliateCatcher`.

Keep the clip small: 100 MB uncompressed on iOS 17 and later for link invocations, less on older targets. Run `npx expo prebuild --platform ios` (or the project's own build) and the type checks. Don't commit generated `ios/` files if the project doesn't keep them.

Then tell the user what only they can do:

- Register the App Group for both bundle IDs in the Apple Developer account.
- Test in Xcode: the App Clip scheme, Run → Arguments → Environment Variables, `_XCAppClipURL` = `https://appclip.apple.com/id?p=<clip bundle id>&fg_click=<click id>&fg_ref=<code>`. Run the clip, then install and open the full app: `useAffiliate()` gives `source: 'app_clip'`. Then call `send_test_event` with that code and click.
- In App Store Connect: upload a build with the clip and create the **default App Clip experience** (header image, subtitle, action). No advanced experience or website is needed.
- Only once the clip is live in the App Store: in the Flow dashboard, step 3 "Your app in the stores" ("Appen i butikerna"), under "Advanced" ("Avancerat"), enter the App Clip bundle ID and choose "App Clip" as the route for iPhones ("Väg för iPhone"). If `get_setup` → `access.tools` lists `update_app` and the user confirms the clip is live, you can do it with `update_app` (`iosAppClipBundleId` and `iosRoute: "app_clip"`); otherwise the user does. Afterwards, `get_setup` shows `appClipInUse: true`.

## 4. Check the App Clip it has

Search the project and report each point as done or missing, with the file:

- The clip target's bundle ID matches `get_setup`'s `appClipBundleId`, and the clip and the full app share an App Group.
- The clip reads the invocation URL with `useLinkingURL()` (not only `Linking.getInitialURL()`, which can miss the App Clip's launch).
- It calls `saveHandoff` with `fg_ref` as `code`, `fg_click` as `clickId` and the `url`.
- It fetches `GET /v1/share/landing/<code>` with a short timeout and shows `greeting` and `bonus`.
- It offers the full app with `displayOverlay()`.
- The full app passes the same storage as `handoff`.
- `expo-updates` is excluded from the clip.

Fix what is missing, then suggest the `_XCAppClipURL` test above.
