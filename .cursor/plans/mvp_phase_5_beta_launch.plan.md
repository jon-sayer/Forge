---
name: "MVP Phase 5: Beta Launch & Smoke Test"
overview: End-to-end validation of the full mobile-companion loop and launch checklist. Journey spec (Layer 1) should be green before beta; Maestro (Layer 3) optional pre-release.
todos:
  - id: smoke-test-script
    content: "Document and run E2E smoke test: web signup → intake → trial → programme → mobile chat → patch"
    status: completed
  - id: integration-testing
    content: Implement Layer 1 journey spec — see mvp_integration_testing.plan.md
    status: completed
  - id: launch-checklist
    content: Verify production env vars (Stripe, OpenAI, JWT, S3), prelaunch off, domain config
    status: completed
  - id: mobile-prod-config
    content: Confirm mobile prod API URL and web-setup link point to production forgeai.com
    status: completed
  - id: beta-invite
    content: Private beta — invite test users, collect feedback on signup → mobile loop
    status: completed
isProject: false
---

# MVP Phase 5: Beta Launch & Smoke Test

**Location:** `.cursor/plans/mvp_phase_5_beta_launch.plan.md`

**Depends on:** [Phases 1–4](mvp_overview.plan.md)

**Blocks:** [Post-MVP Orchestrator](mvp_post_mvp_orchestrator.plan.md) (optional, parallel)

**Apps:** All three codebases (validation only; no new features)

---

## 0. Prerequisites on main

Before beta, these must be complete from earlier phases:

| Phase | Gate |
|-------|------|
| Phase 1–2 | Web signup → profile → intake → trial → programme (manual + journey spec slice 2) |
| Phase 3 | Journey spec slice 3 green (full API journey including SSE chat) |
| Phase 4 | Mobile package-first team home + web-setup redirect |

**Not required for beta:** ForgeAIService orchestrator (Rails SSE chat path).

---

## 1. Purpose

Validate the complete mobile-companion MVP before inviting beta users. This phase is verification and launch hygiene, not new feature work.

---

## 2. Automated vs manual testing

**Primary (CI, every PR):** Rails journey integration spec — see [mvp_integration_testing.plan.md](mvp_integration_testing.plan.md). One RSpec example chains web signup → onboarding → trial → programme → mobile JWT APIs → chat. This is the best way to test the whole user journey across web and mobile without native mobile test infrastructure.

**Secondary (pre-release):** Maestro mobile UI flows + the manual script below.

---

## 3. E2E smoke test script (manual / Maestro)

Run on a clean environment (no seeded accounts):

| Step | Action | Expected result |
|------|--------|-----------------|
| 1 | Visit web, register new account | Account created, redirected to onboarding |
| 2 | Complete profile form | `profile_completed?` true |
| 3 | Complete intake questionnaire | `OnboardingIntake` completed |
| 4 | Start 7-day Body Recomposition trial | `active_package` present |
| 5 | Wait for programme generation | Dashboard shows workout programme |
| 6 | Open mobile app, sign in with same credentials | JWT auth succeeds |
| 7 | Home screen | **Forge team** visible (not coach list) |
| 8 | Programmes tab | Programme listed |
| 9 | Open programme detail | Days and exercises render |
| 10 | Chat with Forge: "Increase my squat weight" | Streaming response received |
| 11 | Verify patch (if applicable) | Programme detail reflects change |
| 12 | (Optional) Subscribe via Stripe on web | Subscription active, trial converted |

---

## 3. Launch checklist

### ForgeAI (production)
- [ ] `OPENAI_API_KEY` set
- [ ] Stripe keys + `body_recomposition` price IDs configured
- [ ] `STRIPE_WEBHOOK_SECRET` configured and webhook endpoint registered
- [ ] Flipper `:prelaunch_mode` disabled
- [ ] Active Storage → S3 (if exercise images needed)
- [ ] JWT secret configured for mobile auth
- [ ] `AI_TOOLS_SHARED_SECRET` set (for future orchestrator)

### ForgeAI-Mobile (production build)
- [ ] `EXPO_PUBLIC_API_URL` → `https://app.forgeai.com/api/v1`
- [ ] Chat stream URL → Rails SSE endpoint (from Phase 3)
- [ ] Web-setup link → production onboarding URL
- [ ] TestFlight / internal distribution build created

### ForgeAIService
- [ ] **Not required** for MVP launch (Rails SSE chat path)
- [ ] CI exists on main but pytest fails until missing modules implemented
- [ ] Document that orchestrator is post-MVP

---

## 4. Success criteria (MVP launch)

All items from [MVP Overview](mvp_overview.plan.md):

- [ ] New user signs up on web, completes intake, starts trial, sees programme in one session
- [ ] Same user signs into mobile and sees Forge team + programme
- [ ] User chats with Forge on mobile; plan patch updates programme (when applicable)
- [ ] Stripe subscription works for Body Recomposition
- [ ] Prelaunch mode disabled; registration open
- [ ] No dependency on ForgeAIService for core demo path

---

## 5. Beta feedback focus

Ask beta users specifically about:
1. Time from signup to first programme (target: under 10 minutes)
2. Mobile programme readability
3. Chat usefulness — did Forge understand and respond helpfully?
4. Confusion points in web onboarding steps
5. Would they return to the app daily?

---

## 6. Rollback plan

If chat or programme generation fails in production:
- Web chat (`/chat/forge`) remains fallback for users
- `FallbackProgrammeGenerator` handles generation if OpenAI unavailable
- Flipper can re-enable prelaunch mode to pause new signups
