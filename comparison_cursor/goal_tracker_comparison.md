# Goal Tracker Codebase Comparison (Bolt vs CreateAnyting vs Replit vs Rork)

## How to Read This Review
- **Focus**: code quality, architecture, scalability potential, backend depth, safety posture, and near-term publishability to mobile app stores.
- **Scope**: functionality completeness was deprioritized per request; emphasis is on technical readiness and maintainability.
- Detailed per-repo notes live in `comparison/repo-notes/`.

## Snapshot Summary
| Repo | Primary Stack | Code Quality | Backend Readiness | Scalability Outlook | Store Readiness | Key Callouts |
| --- | --- | --- | --- | --- | --- | --- |
| Bolt_GoalTracker | Expo 54 + RN 0.81 + Supabase (TS) | ✅ Clean, typed | ✅ Full Supabase schema + RLS | ✅ Managed Postgres + modular UI | ⚠️ Needs bundle IDs/EAS config | Strong foundation; minor release prep |
| CreateAnyting_GoalTracker | Expo 53 (JS) + sandbox tooling | ⚠️ Noisy, generated | ⚠️ Relies on web `/api` without mobile config | ❌ Heavy deps, web bindings | ❌ Native builds break on `window` usage | Must decouple from sandbox & rewrite networking |
| Replit_GoalTracker | React/Vite web + Express/Drizzle (TS) | ✅ Solid web standards | ✅ REST API + Neon/Postgres | ⚠️ Single-user assumptions | ❌ Not a mobile app | Excellent web project, irrelevant for native shipping |
| Rork_GoalTracker | Expo 54 + RN 0.81 (TS) + AsyncStorage | ✅ Polished TS | ⚠️ Backend stubs only | ⚠️ Local-only persistence | ⚠️ Needs API plan + EAS | Closest to publish once backend/story clarified |

Legend: ✅ = strong, ⚠️ = acceptable but needs work, ❌ = blocker.

## Individual Assessments

### Bolt_GoalTracker
- **Strengths**: Thoughtful Supabase integration, typed models, comprehensive SQL migration with RLS + triggers, simple dependency graph.
- **Gaps**: Missing production bundle identifiers and `eas.json`; needs release channel configuration and minor UX polish. Error handling mostly console-based.
- **Scalability**: High confidence—Supabase handles auth/data, schema designed for growth.
- **Store Path**: Add bundle IDs, configure EAS, ensure `.env` provisioning for production keys. No architecture blockers.

### CreateAnyting_GoalTracker
- **Strengths**: Ambitious UI polish, includes sandbox error boundaries, diverse feature scaffolding.
- **Gaps**: Generated sandbox plumbing couples app to browser (`window.parent`, CSS), Expo SDK 53 already sunset, network calls fail off sandbox (`fetch('/api/...')` with no base URL or auth). Large unused dependency surface.
- **Scalability**: Low—maintaining patches and polyfills will be costly; modularization lacking.
- **Store Path**: Requires substantial refactor (upgrade SDK, remove web-only code, design API client, trim dependencies) before builds will even succeed.

### Replit_GoalTracker
- **Strengths**: High-quality TypeScript, proper backend layering (Drizzle + Neon), validated routes, clear React architecture.
- **Gaps**: Entirely web-centric. React Native parity nonexistent; mobile assets/config absent. Backend assumes single default user.
- **Scalability**: Good for web; not applicable to mobile without rewrite.
- **Store Path**: Not viable. Treat as inspiration for service layer, but expect full rebuild to target native platforms.

### Rork_GoalTracker
- **Strengths**: Clean Expo Router implementation, context encapsulates onboarding/journal logic, bundle IDs already set, manageable dependency set.
- **Gaps**: Backend is placeholder; AsyncStorage-only persistence limits multi-device use. TRPC client will throw without `EXPO_PUBLIC_RORK_API_BASE_URL`. Needs security/privacy review for local storage.
- **Scalability**: Moderate—frontend ready, but data tier must be implemented (either remote API or intentional offline story).
- **Store Path**: With a real backend (or explicit offline positioning), plus EAS workflow setup, can reach publishable state relatively quickly.

## Comparative Recommendations
- **Best Launch Candidate**: `Bolt_GoalTracker` — Already integrates managed backend with production-aware schema. Prioritize release engineering tasks (bundle IDs, EAS profiles, QA).
- **Runner-up**: `Rork_GoalTracker` — Choose if you want tighter control over backend stack. Invest effort into implementing/hosting tRPC API or pivot to Supabase/Firebase before release.
- **Needs Major Refactor**: `CreateAnyting_GoalTracker` — Only salvage with deep cleanup (remove sandbox-specific code, define API layer, upgrade SDK, add typings).
- **Out of Scope for Mobile**: `Replit_GoalTracker` — Maintain as a web product or mine for backend ideas, but not a candidate for app store submission.

## Next Steps Checklist
1. **Select Base Repo**: Recommend Bolt for minimal rework; Rork if offline-first is acceptable.
2. **Release Engineering (Bolt/Rork)**:
   - Set `ios.bundleIdentifier` and `android.package`.
   - Add `eas.json` with production profiles; validate Expo SDK compatibility with current release timelines.
   - Provision environment variables securely for CI.
3. **QA & Hardening**:
   - Add user-facing error states for network/storage failures.
   - Implement unit/integration tests for mission-critical flows (Supabase interactions or AsyncStorage serialization).
4. **Security Review**:
   - Double-check Supabase policies (Bolt) or add encryption/privacy messaging for local data (Rork).
5. **App Store Assets**:
   - Prepare store metadata, screenshots, and privacy manifests aligned with whichever base you select.

Refer to the repo-specific notes for deeper dives and targeted remediation ideas.

