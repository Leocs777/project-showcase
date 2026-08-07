# Voyva Case Study

Voyva is a live iOS and web travel planner where the itinerary is edited by talking to it. Say *"Tokyo, four days, with kids"* and a day-by-day plan comes back; say *"swap days five and seven, and make day two lighter"* and the trip rearranges itself. It ships on the App Store and at [app.voyva.app](https://app.voyva.app), with trips synced between them.

Production source is private. This page covers the product and the engineering decisions behind it.

## Product Summary

Most trip planners are forms: pick a date, fill a field, repeat. Voyva's premise is that describing a change out loud is faster than operating a UI, and that this only works if the AI can touch *any* part of an existing trip — not just generate a fresh one and throw away what you had.

- **Build from one sentence** — destinations, pace and interests become a full itinerary with real places, opening hours, and practical notes.
- **Edit by voice or text** — hotels, stops, whole days moved or swapped; hands-free mode reopens the mic after each spoken reply so a conversation keeps going.
- **Natural-voice readback** in English, Chinese and Spanish, female or male, with pause and tap-to-replay.
- **Direct manipulation where it beats talking** — hold a day on the calendar and drag it to a new date.
- **Version history** — every AI change snapshots the trip first, so any edit is one tap from being undone.
- **The rest of a trip**: multi-currency expenses with receipts, per-city photos, offline-first day view, three languages throughout.

## Public URLs

- App Store: https://apps.apple.com/app/id6774875723
- Web app: https://app.voyva.app
- Marketing: https://voyva.app/ · [Privacy](https://voyva.app/privacy) · [Support](https://voyva.app/support) · [Terms](https://voyva.app/terms)

## Architecture

```
iOS (Expo/RN)  ─┐                      ┌─ Groq ─ Gemini ─ Workers AI ─ DeepSeek
                 ├─ Cloudflare Worker ──┤   (ordered chain, first to answer wins)
Web (React/Vite)─┘   quotas · cache     └─ ElevenLabs → PCM gain → WAV
                            │
                     Supabase (auth + trip sync)
```

Every model and voice call goes through one Cloudflare Worker. No API key ever reaches a client, quotas are enforced server-side, and both platforms get identical behaviour from a single prompt implementation.

## Engineering Highlights

### Editing a 53-day trip without blowing the context budget

The first design had the model return the **entire updated itinerary** each turn. It worked for a weekend trip and fell apart on a long one: a 53-day itinerary exceeded the output budget, the JSON truncated mid-array, and the edit failed.

The fix was to stop moving the itinerary through the model at all. The model now returns a small list of **edit operations** — `swap_days`, `remove_stop`, `set_day`, nine in total — which the client applies locally. A bulk edit across 53 days went from unparseable to **27 operations in about 2 seconds**, and output cost dropped roughly tenfold.

Applying them is deliberately forgiving in one direction and strict in the other: an operation referencing a stop that no longer exists is skipped, but if *nothing* in the batch applies the whole turn is rejected rather than silently doing half of what the user asked.

### A provider chain, because free tiers move

Cerebras announced its free tier would convert to paid credits. Rather than swap one hardcoded provider for another, requests now walk an ordered list — Groq → Gemini → Cerebras → Cloudflare Workers AI → DeepSeek — and take the first usable answer. Providers with no key configured are skipped, so adding or retiring one is a secret change with no deploy.

The failure mode this creates is silence: a misconfigured provider is indistinguishable from an absent one, and a retired model name once served zero traffic for a full day without raising a single error. A daily canary now runs one probe through the same code path production uses and records which provider actually answered, exposed at a `/health` endpoint.

### Making synthesized speech audible on a phone

Voice replies were reported as "only loud enough held to your ear." The obvious suspect — the Chinese voice being quieter than the English one — was wrong: measured, they were within 1.5 dB.

Two real causes. First, iOS leaves the audio session in play-and-record after speech recognition, and that session's default output is the **earpiece**, not the speaker. Second, the synthesized audio genuinely sits near −20 dBFS, which is fine on headphones and thin on a phone speaker.

The vendor API has no gain control and MP3 can't be scaled without re-encoding, so the Worker now requests raw PCM, applies gain with a soft-knee limiter, and returns WAV. Measured: **−23.2 → −14.0 dBFS mean, peak −0.3 dBFS, zero clipping**. Audio is cached by voice and text, so a replay costs neither bandwidth nor API credit.

### Cost control on a metered API

Native audio players fetch the same URL two to four times (probe, then ranged reads). Each fetch was triggering a fresh — and separately billed — synthesis. Caching generated audio keyed on voice, model and text made everything after the first request free, and quota counters now increment on generations rather than requests.

## Technical Stack

| Layer | Choices |
| --- | --- |
| iOS | Expo SDK 54 (bare), React Native 0.81, React 19, React Navigation |
| Web | React + Vite, PWA, deployed on Cloudflare Pages |
| Backend | Cloudflare Workers + KV (rate limits, quotas, audio cache) |
| Data | Supabase (auth + sync), AsyncStorage / localStorage offline-first |
| AI | Groq · Gemini · Cloudflare Workers AI · DeepSeek, OpenAI-compatible chain |
| Voice | expo-speech-recognition (on-device STT), ElevenLabs TTS via Worker |
| Payments | RevenueCat (subscriptions, promotional entitlements) |
| Delivery | EAS Update — JS ships over the air across three runtime versions |

## Testing and Delivery

The logic that can silently corrupt a trip is pure and covered by a dependency-free suite (`npm test`, 44 cases): all nine edit operations, the all-or-nothing failure rule, date guarantees, spoken-time rewriting across three languages, and the audio gain stage's ceiling and silence behaviour. Tests load the real source with imports stubbed, so they cannot pass against a copy that has drifted from what ships.

JavaScript changes reach users through EAS Update without an App Store review. Because installs sit on several binary versions, each release publishes to every live runtime rather than only the newest.

## What I Owned

Everything: product direction, iOS and web implementation, the Worker backend and prompt design, the voice pipeline, App Store submissions and review responses, the marketing site, and the release process.

## Engineering Notes

Two lessons worth recording.

**Measure before fixing.** The loudness complaint pointed at the Chinese voice; measurement pointed at audio routing and source level instead. Fixing the reported cause would have shipped a voice change and left the problem in place.

**Resilience hides failure.** A fallback chain means one provider dying is invisible — which is exactly why it needs its own monitoring. Every silent-recovery mechanism deserves a way to ask what it recovered from.
