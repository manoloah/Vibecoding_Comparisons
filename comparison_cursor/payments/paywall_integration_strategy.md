# Paywall Integration Strategy for Mexican Market (Bolt & Replit Codebases)

## Why Third-Party Paywall SDKs Are Required
- Apple and Google mandate native billing for digital goods worldwide (no Stripe for in-app unlocks), including Mexico. External processors are only permitted for physical goods or when routed outside the app experience.[^apple-paywall]
- Paywall middleware (RevenueCat, Qonversion, Adapty) sits on top of StoreKit/BillingClient, abstracting entitlements, subscriber analytics, and cross-platform parity.
- Each platform provides hosted paywall UI, subscription management dashboards, analytics, and webhooks to sync purchases with backend systems.

> **Key implication:** Even though Region = Mexico, billing must still use Apple IAP / Google Play Billing under the hood. The SDK choice determines integration speed, analytics depth, and webhook ergonomics.

[^apple-paywall]: Source: RevenueCat blog “Can you use Stripe for in-app purchases?” (2024) which reiterates that Apple/Google require their billing flows despite antitrust developments.

## Vendor Comparison Snapshot

| Criterion | RevenueCat | Qonversion | Adapty |
| --- | --- | --- | --- |
| **Integration depth** | Mature SDKs for iOS, Android, React Native, Flutter, Unity; official Expo plugin. | Native, React Native, Flutter, Unity. Expo support via React Native bridging. | Native, React Native, Flutter, Unity. Expo integration via React Native package or REST. |
| **Hosted paywalls** | Paywalls 2.0 beta (customizable templates, limited availability). | No-code screens builder with localization & targeting; strong automation tooling.[^qonversionscreens] | Paywall Builder with A/B testing & audience segmentation; focuses on fast iteration. |
| **Entitlement management** | Best-in-class: granular offerings, experiments, intro pricing tracking. | Comparable feature set; adds conversion tracking funnels and marketing automation. | Strong monetization tooling; advanced paywall experiments, churn prediction in higher tiers. |
| **Analytics & dashboards** | Deep subscription analytics, cohort retention, revenue charts; integrates with 3rd parties; free tier up to US$2.5k MTR. | Includes funnels, LTV prediction, revenue attribution; paid tiers scale with revenue. | Emphasizes A/B testing metrics, advanced paywall analytics; pricing tied to MTR tiers. |
| **Webhooks / server events** | Reliable webhooks, but requires custom backend listener (your prior pain point). Also offers integrations (Segment, Amplitude). | Webhooks plus built-in automations (push/email) that can replace some backend work. | Webhooks + Zapier-like automation; optional Adapty Push for in-app paywall updates. |
| **Pricing** | Free ≤ $2.5k MTR, then 1–2% of revenue depending on tier. | Free ≤ $10k MTR, then ~0.8–1.5% effective take depending on plan. | Free ≤ $10k MTR, then 1.2%+ with premium analytics add-ons. |
| **Team maturity** | Widely adopted (Calm, Notion). Rich community support. | Smaller community but praised for targeted automations and customer success. | Positioned for growth teams running paywall experiments; fewer community resources than RevenueCat. |
| **MX compliance considerations** | Relies on store billing—no MX-specific blockers. Supports MX store fronts. | Same as RevenueCat; no-code screens can localize MX pricing messaging quickly. | Same; includes localized pricing & targeting features. |

[^qonversionscreens]: Qonversion Docs — “Getting started with no-code screens”.

### Developer Sentiment (2024–2025)
- **RevenueCat** is still the default pick in community discussions due to documentation, Expo support, and ecosystem integrations (e.g., Slack alerts, Segment). Trade-off: webhooks/ETL must be built yourself.
- **Qonversion** receives positive feedback for marketing automation and paywall editor. Developers note customer support responsiveness and less revenue share for early-stage apps.
- **Adapty** is favored by teams prioritizing A/B testing and paywall iteration speed. Some reports mention smaller SDK footprint and responsive support, but fewer third-party guides.

## Integration Fit by Codebase

### Bolt_GoalTracker (Expo + Supabase)
**Current state:** Expo Router app with Supabase backend, TypeScript, already handling auth via Supabase. No native modules configured yet.

**Recommended SDK:** **RevenueCat** (Expo plugin simplifies setup; built-in Supabase guide for webhooks).

**Rationale:**
- Official Expo config plugin available (`withRevenueCatIOS`, `withRevenueCatAndroid`), reducing native config work.
- Largest knowledge base; quicker to implement given existing Supabase webhook experience.
- Direct Supabase integration examples for syncing entitlements; can reuse RLS policies.

**Implementation Plan (High-Level)**
1. **Prepare Store Products**
   - Create matching product IDs in App Store Connect & Google Play Console (MX localization, pricing tiers in MXN).
   - Define offerings in RevenueCat dashboard (e.g., `monthly`, `annual`, `lifetime`).
2. **Integrate SDK**
   - Install `react-native-purchases` via Expo config plugin.
   - Initialize Purchases in app entry with keys from RevenueCat (prod vs sandbox).
   - Fetch offerings and display paywall (custom UI or RevenueCat Paywalls beta).
3. **Handle Entitlements**
   - On purchase success, call Supabase RPC/webhook endpoint to sync entitlement state to your Postgres tables.
   - Set up RevenueCat webhook for `INITIAL_PURCHASE`, `CANCELLATION`, `EXPIRATION` events; update Supabase `user_stats`/`user_themes` accordingly.
4. **UX Polish**
   - Gate premium features in app via `customerInfo.entitlements`.
   - Add restore purchases button (mandatory).

**Effort Estimate:** 2–3 engineering days (assuming store configs ready), plus webhook endpoint (1 day) and QA (1 day). Your prior webhook pain can be mitigated by reusing or templating the Supabase edge function.

### Replit_GoalTracker (React web + Express)
**Current state:** Web-first project; no mobile binary. Would need full React Native/Expo rewrite for parity. Backend uses Express + Drizzle + Neon.

**Feasibility Assessment:**
- As-is, cannot integrate StoreKit/BillingClient because there is no native shell. You’d first need to port to Expo/React Native (likely weeks of work).
- Backend already has session handling; could host RevenueCat/Qonversion/Adapty webhook endpoints easily.

**After React Native Port:** prefer **Qonversion** if marketing automation or no-code paywalls valuable during rebuild. Otherwise RevenueCat remains safer due to ecosystem.

**Implementation Blockers:**
- Need to rebuild UI with React Native components.
- Set up native projects, provisioning, and release pipelines.
- After port, integration steps similar to Bolt (SDK init + webhook). Express server can receive purchase events.

**Effort Estimate:** Porting alone 4–6 weeks+; paywall integration 3–4 days once mobile foundation exists.

### Which Codebase Is Easier?
`Bolt_GoalTracker` is dramatically easier:
- Already mobile-native with Expo; minimal native customization history.
- Supabase backend can ingest RevenueCat/Qonversion/Adapty webhooks quickly.
- Replit repo requires a full platform migration before any paywall SDK becomes usable.

## Decision Matrix by Scenario

| Scenario | Suggested SDK | Notes |
| --- | --- | --- |
| “Ship ASAP on Bolt” | RevenueCat | Expo plugin, broad docs, webhook familiarity. |
| “Rebuild Replit web app as mobile + launch paywalls” | RevenueCat initially; evaluate Qonversion later | RevenueCat fastest to unblock launch; Qonversion’s no-code screens help marketing iterate post-MVP. |
| “Need advanced A/B testing & paywall experiments” | Adapty | Strongest paywall builder; ensure engineering time for custom paywall UIs. |
| “Want marketing automation + no-code paywalls from day 1” | Qonversion | Automation suite reduces need for bespoke webhook workflows. |

## Migration Strategy if You Switch Vendors Later
1. Keep product IDs identical across stores to avoid forced repurchase.
2. Implement RevenueCat across Android + iOS + Web (optional via Stripe).
3. Extend to additional segments with no-code paywalls and automation if/when needed.

## Recommended Path Forward
1. **Adopt RevenueCat for Bolt today.** Fastest Expo integration, robust analytics, and existing familiarity outweigh webhook maintenance.
2. **Plan a native rewrite for Replit before worrying about paywalls.** Decide whether to fork Bolt or rebuild with shared component library; once mobile, reuse the RevenueCat integration pattern.
3. **Re-evaluate Qonversion or Adapty** only if marketing automation (Qonversion) or deep paywall testing (Adapty) become pressing needs that RevenueCat cannot cover.

## Risks & Mitigations
- **Webhook reliability:** Host webhooks on Supabase Edge Functions (Bolt) or Express (Replit). Add retry/backoff, idempotency keys, and audit logs.
- **MX pricing & taxation:** Ensure App Store / Play Console local currency pricing and VAT settings align; test introductory and upgrade flows in sandbox MX storefronts.
- **Regulatory disclosures:** Update privacy policy and in-app disclosures to reference Apple/Google billing plus data sharing with chosen SDK.
- **Platform review:** Follow App Store / Play policies for paywalls (restore purchases button, link to EULA, cancellation instructions).

## Implementation Checklist (Bolt First)
- [ ] Configure App Store & Play Console products in MXN with localized display names.
- [ ] Add RevenueCat SDK via Expo config plugin and initialize with sandbox keys.
- [ ] Build paywall screen (custom or hosted) and entitlement gating logic.
- [ ] Deploy webhook endpoint (Supabase Edge or serverless function) listening for RevenueCat events.
- [ ] Instrument analytics to measure conversion (RevenueCat charts + Supabase events).
- [ ] Add localized MXN pricing & messaging in paywalls.
- [ ] Regression-test entitlement gating across login/logout and fresh installs.
- [ ] Document webhook payloads and retry logic for incident response.

---

*Prepared for Goal Tracker comparative study — November 2025.*

