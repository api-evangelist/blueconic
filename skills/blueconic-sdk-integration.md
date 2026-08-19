---
name: blueconic-sdk-integration
description: Integrate, instrument, verify, or audit BlueConic SDKs for Android, iOS, React Native, Flutter, and browser-based Web customer apps. Use when an AI coding assistant is asked to add SDK dependencies, initialize BlueConic, track PAGEVIEW or app events, wire dialogues/listeners/recommendations/geofencing, review privacy and consent handling, run build verification, create customer integration guidance, or audit an existing BlueConic SDK setup.
---

# BlueConic SDK Integration

## Overview

Use this skill as the canonical BlueConic SDK integration guide. Keep tool-specific agent, Cursor, and AGENTS.md files as generated wrappers; do the actual SDK work from this skill and the platform reference files.

## Choose The Platform

Identify the host app platform from repository files and the user's request:

- Android native: read `references/android.md`.
- iOS native: read `references/ios.md`.
- React Native: read `references/react-native.md`.
- Flutter: read `references/flutter.md`.
- Web SDK for browser-based app experiences: read `references/web.md`.

If the platform is ambiguous, inspect the repository before asking. Common signals are `build.gradle` or `settings.gradle` for Android, `.xcodeproj` or `Package.swift` for iOS, `package.json` with React Native dependencies, `pubspec.yaml` with Flutter dependencies, and `package.json` with `@blueconic/blueconic-web-sdk` or browser app tooling for Web. For tag-loaded websites where BlueConic is already present on the page, use the JavaScript front-end API rather than the Web SDK.

## Workflow

For full customer integrations, follow this order:

1. Integrate: dependency setup, initialization, simulator support, and first PAGEVIEW.
2. Track journeys: PAGEVIEW coverage, app events, timeline events, recommendations, and listener publish events.
3. Wire dialogues/listeners: properties dialogues, recommendations, custom event streams, plugins, and geofencing.
4. Verify build: package, native, static API, and runtime smoke checks.
5. Review privacy: consent, profile data, permissions, debug/simulator, and policy files.
6. Audit: read-only final pass with findings ordered by severity.

Within those workflows, cover advanced SDK APIs when the customer use case needs them: profile ID access, profile get/set/add/increment/update, local or server-side profile reset, consent objectives, locale, segment reads, custom selector events, listener context events, simulator wiring, CTV support, browser CSP/storage constraints, and native platform privacy files.

Use only the sections needed for the user's request. For example, if the user asks for a privacy review, read the platform reference and start from the privacy section.

## Operating Rules

- Prefer the host app's existing architecture, dependency manager, navigation layer, environment config, and analytics wrapper.
- Collect missing tenant values as placeholders. Do not invent host names, profile property IDs, objective IDs, content store IDs, or app IDs.
- Initialize BlueConic before any SDK calls.
- For Web SDK integrations, confirm the app runs in a browser environment with `window`, `document`, `fetch`, and `localStorage`; do not install the Web SDK into ordinary websites that should use the BlueConic tag or JavaScript front-end API.
- Use stable, non-PII screen names and event names.
- Keep debug, simulator, and verbose logging out of production paths.
- Do not log profile IDs, personal data, consent strings, full dialogue payloads, or recommendation payloads in production.
- Run the lightest useful verification first, then broaden when native files or shared flows changed.
- When unsure about an SDK API, consult the platform reference first, then the official BlueConic documentation linked from that reference, then the installed SDK source or package.
- If SDK documentation and installed SDK source disagree, trust the installed source for compiling code and report the mismatch.
- For initialization, profile updates, event publishing, and native bridge calls, use the callback, promise, suspend, or async style that best matches the host app and handle failures explicitly.
- For less common APIs, inspect the installed SDK package or source before generating code. Prefer a compilable implementation with a reported docs/source mismatch over copying stale documentation snippets.

## Output Expectations

When making changes, report:

- Files changed with line references.
- SDK version and dependency manager used.
- Host/app ID placeholders that still need customer values.
- Screens, events, profile properties, consent objectives, and listener/dialogue hooks added.
- Native Android/iOS changes such as manifest entries, ProGuard/R8, URL schemes, pods, privacy manifests, permissions, or ATT/IDFA handling.
- Web changes such as package manager updates, route tracking hooks, CSP/connect/resource domain configuration, debug/simulator gates, and browser storage assumptions.
- Advanced SDK APIs touched, including profile merge/reset, locale, segments, custom selectors, listener context events, simulator settings, and CTV constraints.
- Verification commands run and skipped commands with reasons.

When auditing, lead with findings ordered by severity and include file/line references, impact, recommended fix, and tenant confirmations needed.
