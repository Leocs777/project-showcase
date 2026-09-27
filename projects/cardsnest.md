# CardsNest Case Study

CardsNest is a credit-card discovery and wallet-tracking app for iOS and Android. Version 1.4.2 is live on the App Store; the 1.4.3 iOS release candidate has been uploaded to App Store Connect and is being prepared for review. The production source repository remains private; this page describes the product and engineering work without mirroring private source or release materials.

## Product Summary

CardsNest helps people compare U.S. credit cards and track the benefits on cards they already hold. The current live version keeps wallet records on the device without requiring an account. The 1.4.3 release candidate adds optional Apple or Google sign-in to sync selected wallet records across devices. CardsNest does not connect to bank accounts.

## Public URLs

- App Store: https://apps.apple.com/app/id6775310510
- Marketing: https://cardsnest.tmslabs.net/
- Privacy Policy: https://cardsnest.tmslabs.net/privacy
- Support: https://cardsnest.tmslabs.net/support
- Terms: https://cardsnest.tmslabs.net/terms

## Technical Stack

- SwiftUI iOS app and a native Java Android client
- Local-first guest storage, with optional Cloudflare Worker and D1 account sync
- Apple and Google sign-in with server-verified identities and device-bound handoff
- English, Simplified Chinese, Traditional Chinese, and Spanish interfaces
- Static marketing and support site deployed with Cloudflare Pages
- Versioned public card and news feeds kept separate from private wallet records

## Implementation Highlights

- Built a searchable catalog of 185 credit cards with issuer, product, tier, and availability filters, plus side-by-side comparison for up to three cards.
- Created wallet tracking for card dates, annual-fee recovery, recurring credits, editable estimated values, free-night awards, companion certificates, bonus goals, and usage history.
- Kept guest use available without registration; added optional account sync, portable backups, sign-out, and account deletion in the 1.4.3 release candidate.
- Added revision-aware per-record wallet merging so independent changes can sync without replacing an entire device snapshot in the 1.4.3 release candidate.
- Bundled authentic card artwork where available and retained a local fallback for image failures.
- Built public marketing, privacy, support, terms, and data-feed surfaces for the App Store product.

## Privacy And Reliability Decisions

The app never asks for bank credentials or full card numbers. Guest records remain on the device. When a user enables account sync, the service receives the sign-in email and selected wallet records over HTTPS; synced data is not end-to-end encrypted. Sign-in credentials are handled by Apple or Google, and provider passwords are not sent to CardsNest. Users can export a backup, sign out, and request account deletion.

Public catalog and news updates use a versioned feed kept separate from account data so content changes can remain compatible with released clients. Benefit values and eligibility can change, so the app directs users to verify current terms with the card issuer.

## Verification

The iOS release candidate was archived and uploaded to App Store Connect. Android API 36 emulator checks cover catalog, tracking, navigation, persistence, import, and OAuth callback handling. The account service has automated checks for identity validation, replay protection, revisions, deletion, and portable records. A full native cross-platform sync walkthrough remains a separate release-validation item.

## What I Owned

- Product scope, information architecture, and localization
- iOS and Android clients, wallet data model, and cross-device merge behavior
- Optional account service and public marketing/support site
- Card-data quality workflow, App Store materials, and release operations

## Safe Review Notes

The private production repository is not mirrored here. This case study excludes source code, credentials, account and team identifiers, customer data, local build artifacts, private commit history, and unpublished operational configuration.
