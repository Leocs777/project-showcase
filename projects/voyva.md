# Voyva Case Study

Voyva is a private Expo / React Native app project with an independent public App Store brand and support site. The production source remains private; this page summarizes the visible product and engineering scope.

## Product Summary

Voyva is structured as a mobile app with a separate static brand site for App Store launch requirements. The repository includes an Expo app foundation and a standalone `brand-site` directory so public web pages can deploy independently from the native app source.

## Public URLs

- Marketing: https://voyva.app/
- Privacy Policy: https://voyva.app/privacy
- Support: https://voyva.app/support
- Terms: https://voyva.app/terms

## Technical Stack

- Expo 54
- React 19 and React Native 0.81
- React Navigation bottom tabs
- AsyncStorage for local persistence
- Expo localization and `i18n-js` for localization support
- Expo Calendar, FileSystem, ImagePicker, Print, Sharing, SVG, and UI utility modules
- PostHog React Native dependency available for analytics integration
- Static brand site deployed separately through Cloudflare Pages

## Implementation Highlights

- Maintained a mobile app codebase and launch website as separate surfaces inside the same product repository.
- Prepared App Store support URLs for marketing, privacy, support, and terms pages.
- Used a React Native / Expo stack suitable for rapid cross-platform iteration.
- Kept the brand site independent from app source, native modules, secrets, and local configuration.

## Engineering Notes

The separation between the app and `brand-site` lowers launch risk: Cloudflare Pages can deploy legal/support pages without requiring native build tooling, while the app code can remain private and evolve independently.

## What I Owned

- Product launch website structure
- Cloudflare Pages deployment model
- App Store URL readiness
- Mobile app foundation and dependency choices
- Repository separation between public web assets and private app implementation

## Safe Review Notes

The private production repo is not mirrored here. This case study describes the architecture and deployment pattern without exposing app source or private history.
