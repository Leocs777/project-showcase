# CardsNest Case Study

CardsNest is an iOS app for browsing, comparing, and locally tracking U.S. credit cards. The production source repository remains private; this page summarizes the product and engineering work in a recruiter-friendly format.

## Product Summary

CardsNest helps users compare credit cards, track cards they own, manage annual fee awareness, and follow statement-credit usage. The app is designed around local-first personal finance tracking: it does not require a user account and does not connect to bank accounts.

## Public URLs

- Marketing: https://cardsnest.tmslabs.net/
- Privacy Policy: https://cardsnest.tmslabs.net/privacy
- Support: https://cardsnest.tmslabs.net/support
- Terms: https://cardsnest.tmslabs.net/terms

## Technical Stack

- SwiftUI iOS app
- Local persistence with `UserDefaults`
- Multilingual app experience: English, Simplified Chinese, Traditional Chinese, and Spanish
- Static App Store support site deployed through Cloudflare Pages
- GitHub-backed deployment workflow for public legal/support pages

## Implementation Highlights

- Built a browsable card catalog with 179 credit card entries.
- Added search, filtering, and comparison workflows for up to three cards.
- Implemented a local wallet tracker with card nicknames, open dates, annual fee awareness, notes, and statement-credit checklist state.
- Added graceful image fallback behavior when remote card art cannot load.
- Designed App Store-ready support surfaces: marketing page, privacy policy, support page, terms, robots.txt, sitemap.xml, and deployment README.
- Kept sensitive wallet data local and separated public web assets from app source.

## Privacy And Compliance Decisions

CardsNest is intentionally built without bank-account linking. The public privacy policy explains that wallet entries, card nicknames, last-four digits if entered, open dates, notes, and preferences are stored locally on device. Anonymous analytics are opt-in and disabled unless a backend is configured.

The terms page includes financial disclaimer language, referral-link disclosure, issuer independence, and offer-accuracy caveats appropriate for a credit card comparison tool.

## What I Owned

- Product naming and App Store support URL strategy
- iOS feature scope and data model decisions
- Multilingual support strategy
- Privacy and terms drafting for launch readiness
- Static brand site build, Cloudflare Pages deployment, DNS setup, and HTTPS validation

## Safe Review Notes

The private production repo is not mirrored here because it may contain full source, historical commits, internal implementation details, and App Store launch materials. Sanitized code samples or a guided walkthrough can be shared separately when appropriate.
