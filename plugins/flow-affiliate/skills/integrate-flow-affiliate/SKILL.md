---
name: integrate-flow-affiliate
description: Integrate Flow Affiliate (affiliate and publisher tracking) into an Expo or React Native app, its Node or other backend, or a website, with the Flow Affiliate MCP tools, then test it against Flow's test environment. Use when the user asks to add or integrate Flow Affiliate, Flow's SDK (@flow/expo, @flow/growth-sdk), or Flow's affiliate or referral tracking.
argument-hint: "[app | backend | web]"
---

# Integrate Flow Affiliate

Flow Affiliate credits a sign-up to the publisher whose link brought the user. Two parts do it:

- **The app** catches the publisher's code and click with `@flow/expo` (on first launch it asks Flow for the click behind the install) and sends them to its own backend with the sign-up.
- **The backend** confirms the sign-up to Flow, server to server, with a signed `user.registered` event carrying `clickId` (fg_click) and `affiliateCode` (fg_ref). Money only moves after that confirmation, and only in production.

Everything here runs in Flow's **test environment**. The Flow tools never touch production events or move money. A key can also be allowed to change the program's setup (and, separately, its payouts and terms): `get_setup` → `access` says what this key can change now. Those changes are made as the person who created the key, apply at once, and the user is asked before each one.

Optional argument (`$ARGUMENTS`): `app`, `backend` or `web` to do only that part.

## 0. Check the tools

Call the `get_setup` tool of the `flow-affiliate` MCP server. If the tool is missing or answers 401:

- Tell the user to create an AI key in the Flow dashboard: their program, step 1 "Install the SDK" ("Installera SDK:t"), "Let your AI assistant do it" ("Låt din AI-assistent göra det"), then `export FLOW_AI_KEY=fg_ai_…` and restart the assistant.
- Stop until it works. Never guess the app id or URLs.

## 1. Read the program

From `get_setup`, note the app id, the API URL, where conversions happen (`convertsIn`), the iPhone route, the steps still to do and `access` (what this key may change). Tell the user in two or three lines what you found.

## 2. Find the parts of the project

Inspect the working directory before changing anything. Use the first rule that matches for each part, and say which files gave it away:

1. **Expo app**: `package.json` has `expo` (and usually `app.json` or `app.config.*`). Target `expo_app`.
2. **Bare React Native app**: `package.json` has `react-native` but not `expo`, with `ios/` and `android/`. Target `react_native_app`.
3. **Node backend**: a server `package.json` with `express`, `fastify`, `@nestjs/core`, `hono`, `koa`, Next.js API routes or similar. Target `node_backend`.
4. **Other backend**: Python, PHP, Ruby, Go, Java, .NET and so on. Target `other_backend`.
5. **Website** with its own sign-up: Target `web`, plus its backend.

A monorepo can have several parts; do each. If the app and its backend are in different repositories, do the one that is here and say what the other needs. If nothing matches, ask the user.

## 3. The app

1. Call `get_integration_guide` with the app's target and follow it exactly: the install command, `AffiliateProvider` with `installMatch` around the root, and the sign-up sending `affiliate.code` (as `affiliateCode`) and `affiliate.clickId` (as `clickId`) to the app's own backend.
2. `@flow/expo` and `@flow/sdk-core` are not on the public npm registry yet. Ask the user for the package files (tarballs) from their Flow Group contact, put them in the project (for example `vendor/flow-sdk/`) and install them from there. Never install a package of that name from npm.
3. Fit the code to the app: Expo Router (`app/_layout.tsx`) or a root `App.tsx`; the app's own sign-up call and form. Keep the code field at sign-up if there is one.
4. iPhone: call `app_clip_guide`. If the program sends iPhones to an App Clip, the full app must pass `handoff` to `createAffiliateCatcher`: do it now. If it has none, mention in one line that an App Clip can greet invited people and hand the click over without a code, and offer the `add-app-clip` skill. Don't add one unasked.

## 4. The backend

1. Call `get_integration_guide` with `node_backend` or `other_backend`.
2. Find where the backend creates a user. After the user is saved, send `user.registered` with `eventId` `registered:<user id>`, `appUserId` (the backend's own user id, never an email address or phone number), and the `affiliateCode` and `clickId` the app sent. Accept both fields in the sign-up request. The field is `affiliateCode`: Flow refuses `referralCode` with `400 renamed_field`.
3. Read the key from the environment: `FG_KEY_ID` and `FG_SECRET`. You cannot create it: ask the user for a **test key** from the dashboard, step 2 ("Create test key" / "Skapa testnyckel"). Add the names to `.env.example`; never write a real secret into code, git or the app.
4. A Flow error must never fail the user's sign-up: the Node SDK resolves to `{ ok: false }`; in other languages, catch and log.
5. Other events the program pays on: call `list_events` and send only those the app really has (for example `purchase.completed` with `amount` in cents and `currency`). Reversals name the `reversesEventId` they undo.

## 5. Fill in the setup (only if `access.tools` lists the tool)

1. Store links and ids with `update_app`: the bundle ID from `ios.bundleIdentifier` in `app.json`/`app.config.*` (or `PRODUCT_BUNDLE_IDENTIFIER` in the Xcode project) with the Team ID from `ios.appleTeamId` (or `DEVELOPMENT_TEAM`) as `iosAppIds: ["TEAMID.bundle.id"]`; the Android package from `android.package` or `applicationId` in `android/app/build.gradle`. Store links only as the user gave or confirmed them: ask for the App Store and Google Play URLs, never build one from an id.
2. Events the backend really sends beyond the standard ones: `add_event` with a name in Swedish and English.
3. A logo or material image the user points to: `upload_file`, then run the `curl` command it answers.
4. Payouts, budgets and terms (`add_payout_line`, `update_offer`, `save_terms`) only with amounts and text the user gave you. Never guess them.
5. Never invent an identifier, link or domain. After each change, tell the user in one or two lines what changed. If a tool says the key or its creator may not, tell the user what to do in the dashboard instead.

## 6. Test

1. Call `send_test_event` (no arguments: a test sign-up). Tell the user what came back, including whether step 2 ticked. Explain `reason` and `meaning` if it was ignored: a program that is not live yet credits nobody, and that is expected.
2. If the backend runs locally and the user has put the test key in its environment, make a test sign-up through it (its own test, a script or a curl to the local server), then call `check_status`. Fix what it shows: a wrong signature means the bytes signed differ from the bytes sent, or the secret belongs to another key.
3. Run the project's own type checks and tests.

## 7. Finish

Call `get_setup` again and report:

- What you changed, file by file, in a few lines, and what you changed in Flow.
- What the user must still do: the test key in the backend's environment if not done; the dashboard steps still to do (store links, payouts, material) and "Send for review" ("Skicka för granskning"); after approval, a production key for the backend.

Never say the integration is live or paying: only Flow Group's approval and production events do that.
