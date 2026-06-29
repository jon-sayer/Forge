---
name: "Post-MVP: Multi-Agent Orchestrator"
overview: Repair ForgeAIService to boot, align patch schema with Rails, and restore multi-agent coaching. ~35% done on main — CI + scaffold exist; 4 core modules still missing, service cannot boot.
todos:
  - id: missing-modules
    content: "Implement auth.py, clients/rails_tools.py, observability/run_logger.py, schemas/proposal.py"
    status: pending
  - id: patch-translation
    content: "Translate JSON Patch specialist output to Rails domain ops (set_exercise_params, swap_exercise, etc.)"
    status: pending
  - id: auth-bridge
    content: "Bridge mobile JWT to orchestrator auth (validate via Rails or proxy through Rails)"
    status: pending
  - id: orchestrator-tests
    content: "Integration tests for POST /chat with mocked OpenAI + Rails"
    status: pending
  - id: orchestrator-deploy
    content: "Production Dockerfile, deploy orchestrator.forgeai.com (CI already exists on main)"
    status: pending
  - id: mobile-switch
    content: "Optional — point mobile stream.ts back to orchestrator once stable"
    status: pending
isProject: false
---

# Post-MVP: Multi-Agent Orchestrator

**Location:** `.cursor/plans/mvp_post_mvp_orchestrator.plan.md`

**Depends on:** [Phase 5](mvp_phase_5_beta_launch.plan.md) (MVP launched with Rails SSE chat)

**App:** ForgeAIService (with Rails AI Tools API integration)

**Not required for MVP launch.**

---

## 0. Already on main (do not rebuild)

| Item | Status |
|------|--------|
| `.github/workflows/ci.yml` (ruff + pytest) | Done |
| `RAILS_TOOLS_BASE_URL` in `app/config.py` | Done |
| Orchestrator scaffold (`orchestrator.py`, `merge.py`, `registry.py`, chat endpoint) | Done |
| README with env vars + SSE contract | Done |
| Deterministic plan engine (`/v1/plan/*`) | Done |

**Still missing:** `auth.py`, `clients/rails_tools.py`, `observability/run_logger.py`, `schemas/proposal.py` — service cannot import/boot via `app.main:app`.

---

## 1. Why post-MVP

The orchestrator provides multi-agent differentiation (Router → Forge + Nova + Fuel specialists → merged patches). It is substantial work and currently **cannot boot**. MVP ships mobile chat via Rails SSE ([Phase 3](mvp_phase_3_rails_sse_chat.plan.md)), which reuses the working `CoachChatService`.

---

## 2. Current blockers

| Issue | Detail |
|-------|--------|
| Missing modules | `auth.py`, `clients/rails_tools.py`, `observability/run_logger.py`, `schemas/proposal.py` |
| Patch schema mismatch | Orchestrator uses JSON Patch; Rails `PlanPatchApplier` expects domain ops |
| Auth mismatch | Mobile sends JWT; service expects `X-Internal-Api-Key` |
| Missing `programme_id` | Patch tool calls need programme context from `get_current_plan` |
| Stale README | Agent names (Stride, Velo) don't match current registry |
| No CI/Dockerfile | No production deploy path |

---

## 3. Implementation phases

### 3a. Boot the service
- Implement 4 missing modules following README/AGENTS.md contracts
- Fix config: unify `RAILS_TOOLS_BASE_URL` vs `RAILS_URL`
- All existing pytest files pass

### 3b. Align Rails integration
- `RailsToolClient` HMAC signing (match [`AiToolsAuthentication`](ForgeAI/app/controllers/concerns/ai_tools_authentication.rb))
- Patch translation layer: JSON Patch → `{ op: "set_exercise_params", ... }`
- Include `programme_id` from `get_current_plan` in patch/swaps
- Fix or remove stale tools (`log_checkin`, `log_workout`, `regenerate_programme`)

### 3c. Auth for mobile
Options (pick one):
1. **Rails proxy** — mobile keeps calling Rails; Rails forwards to orchestrator with internal key
2. **JWT validation** — orchestrator validates JWT by calling Rails `/api/v1/me`
3. **Dual auth** — accept either internal key (server) or JWT (mobile)

### 3d. Production readiness
- Production Dockerfile
- ~~CI: ruff + pytest on push~~ — **done on main**
- Deploy to `orchestrator.forgeai.com`
- Health check with Rails connectivity probe

### 3e. Mobile migration (optional)
- Feature flag or env switch to point `stream.ts` at orchestrator
- A/B test multi-agent vs single-coach responses

---

## 4. Success criteria

- [ ] `uvicorn app.main:app` boots without import errors
- [ ] `pytest -v` passes (currently fails on import due to missing modules)
- [ ] `POST /chat` streams SSE with mocked OpenAI + Rails
- [ ] Plan patches applied via Rails AI Tools API with correct domain ops
- [ ] Deployed to production; mobile can optionally connect
- [ ] Run logs written to stdout (and optionally SQLite in dev)

---

## 5. Files (primary)

### New (ForgeAIService)
- `app/auth.py`
- `app/clients/rails_tools.py`
- `app/observability/run_logger.py`
- `app/schemas/proposal.py`
- `tests/test_chat_integration.py`
- `Dockerfile`

### Modified
- `app/agents/orchestrator.py` (patch translation, programme_id)
- `app/config.py`
- `README.md`
