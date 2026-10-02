# Dhyaan — 3-Person Work Distribution, Integration & Execution Plan

| | |
|---|---|
| **Covers** | Document 2 (work distribution & execution plan) **and** Document 3 (integration, collaboration & success framework), plus the consolidated roadmap and the one-page "Who Does What" |
| **Product spec** | `PRD.md` (requirements, architecture, APIs, data, rules, TBD register). This file does **not** repeat requirements; it links to PRD sections like `PRD §7.1` |
| **External API fields** | `connector-api-specs.md` |
| **Platform** | Pine Labs AgenticOrg — https://agenticorg.ai/dashboard |
| **Hard deadline** | **Sun 4 Oct 2026, 11:59 PM IST** (target submit 11:00 PM; 1 h buffer) |
| **Version** | 1.0 · 3 Oct 2026 |

---

## How to use this document
1. **Everyone** reads Part A (shared contracts) and the one-pager (Part E) first.
2. Each person then works from **their own section** in Part B, in task order.
3. Part C describes how we merge, test on AgenticOrg and decide we're done.
4. Part D is the master timeline. When in doubt, Part D wins.

## Team at a glance
| | Person 1 — Experience (Frontend/UI) | Person 2 — Agent & APIs (Backend) | Person 3 — Data, Infra & QA |
|---|---|---|---|
| **Mission** | Everything a human sees, hears, taps or reads | Everything the agent thinks, decides and calls | Everything that stores, deploys, resets, logs and proves it works |
| **Owns** | Telegram bot UX, message templates, approval cards, Family View tab, voice-reply UX, demo script, persona roles | AgenticOrg agent + system prompt, connectors, mock-server **routes & business logic**, policy rules, audio pipeline | GitHub repo, Vercel + Neon, **DB schema/repo/seed**, scenario engine + admin API, Sheets workbook, CI, eval runs, run log, E2E |
| **Main "UI"** | Telegram + Google Sheets Family View | AgenticOrg dashboard | Terminal / Bruno / Sheets / Vercel |

```mermaid
flowchart LR
    P1["Person 1<br/>Experience"] -- "templates + callback_data spec" --> P2["Person 2<br/>Agent and APIs"]
    P2 -- "decision points + variables" --> P1
    P3["Person 3<br/>Data, Infra, QA"] -- "repo + scenario engine + URL + Sheets" --> P2
    P2 -- "OpenAPI + repo needs + rule IDs" --> P3
    P3 -- "workbook tabs + seed" --> P1
    P1 -- "persona chat IDs + demo script" --> P3
    P2 -- "connectors + agent" --> AO["AgenticOrg"]
    P3 -- "eval runs + E2E" --> AO
```

---

# PART A — Shared Contracts (frozen at the end of Phase 1)

Nobody changes anything in Part A after the freeze without the process in A10.

### A1. Project / folder structure (with owner)
```
Ken_case_comp/                         (GitHub repo — created by P3)
├── docs/                              all  — PRD.md, WORK_DISTRIBUTION.md, connector-api-specs.md
├── contracts/                         SHARED — change only via A10
│   ├── openapi/delhivery-mock.yaml    P2 writes, P3 reviews
│   ├── openapi/pinelabs-mock.yaml     P2 (only if mocking — TBD-P4)
│   ├── openapi/extras.yaml            P2 (≤3 extra capabilities — TBD-D4)
│   ├── openapi/admin.yaml             P3 writes, P2 reviews
│   ├── schemas/*.json                 P2 + P3 (A6)
│   ├── sheets-schema.md               P3 writes, P2 reviews
│   ├── db-repo-interface.md           P3 writes, P2 reviews (A5)
│   ├── rule-ids.md                    P2 writes, P3 reviews (A7)
│   ├── message-templates.md           P1 writes, P2 reviews
│   └── callback-data.md               P1 + P2 (A6)
├── mock-server/                       Node 20 + Express, deployed to Vercel
│   ├── api/index.js                   P3 (Vercel entry)
│   ├── src/app.js                     P2 (route mounting)
│   ├── src/routes/delhivery/          P2
│   ├── src/routes/pinelabs/           P2
│   ├── src/routes/extras/             P2
│   ├── src/services/                  P2 (business logic)
│   ├── src/middleware/auth.js         P2
│   ├── src/middleware/errors.js       P2
│   ├── src/middleware/logging.js      P3
│   ├── src/scenarios/                 P3 (engine)
│   ├── src/admin/                     P3 (admin API)
│   ├── src/db/                        P3 (schema.sql, migrations/, seed.sql, repo.pg.js, repo.memory.js, index.js)
│   ├── tests/unit/  tests/api/        P2 (routes/services) · P3 (db/scenarios/admin)
│   ├── tests/contract/                P3
│   ├── vercel.json  package.json      P3
│   └── .env.example                   P3
├── agent/
│   ├── system-prompt/v1.md, v2.md …   P2 (immutable once used in a run)
│   ├── system-prompt/CHANGELOG.md     P2
│   └── platform-notes.md              P2 (connector names, settings, TBD answers — no secrets)
├── sheets/
│   ├── setup.md                       P3 (workbook ID, sharing, validations)
│   └── family-view.md                 P1 (formulas/formatting notes)
├── evals/
│   ├── cases.md                       P3 + P2 (the 10 cases + reserve)
│   ├── runs/round-01.md …             P3
│   └── still-failing.md               P3 + P2
├── demo/
│   ├── story.md                       P1 (100-word story)
│   ├── recording-script.md            P1
│   └── decision-table.md              P3 + P2 (export of decision_log for Part 1 Q2)
├── submission/answers.md              all (A — ownership in C10)
└── .github/ (workflows/ci.yml, pull_request_template.md, CODEOWNERS)   P3
```

### A2. Naming conventions
| Thing | Convention | Example |
|---|---|---|
| Connector names (org-wide unique) | `<tool>_dhyaan` (suffix **TBD-D7**) | `gnani_dhyaan`, `delhivery_dhyaan`, `telegram_dhyaan` |
| Sheets tabs & columns | `snake_case` | `care_tasks.depends_on` |
| IDs | `PREFIX-NNNN` | `MEM-001`, `PAT-001`, `RX-001`, `JRN-0003`, `TSK-0012`, `APR-0007`, `ORD-0003`, `EVT-0101`, `DEC-0042` |
| Delhivery `order` field | = our `order_id` | `ORD-0003` |
| Enums | `UPPER_SNAKE` | `AWAITING_APPROVAL` |
| Timestamps | ISO 8601 with `+05:30` | `2026-10-04T10:15:00+05:30` |
| Money | integer rupees, `_inr` suffix | `total_inr: 1850` |
| Rule IDs | `R-<AREA>-NN` | `R-POL-02` |
| Template IDs | `T-<NAME>` | `T-APPROVAL-REQUEST` |
| Eval cases / runs | `E01`…`E10` / `RUN-<round>-<case>` | `RUN-02-E04` |
| Prompt versions | `v<N>` | `agent/system-prompt/v3.md` |
| JS code | camelCase functions, kebab-case files | `create-shipment.js` |
| Branches | `p1/…`, `p2/…`, `p3/…`, `contract/…` | `p2/delhivery-tracking` |

### A3. Environment variables
| Variable | Used by | Created by | Stored in | Example / note |
|---|---|---|---|---|
| `REPO_DRIVER` | mock-server | P3 | Vercel + local `.env` | `pg` (prod) / `memory` (local dev, tests) |
| `DATABASE_URL` | mock-server | P3 | Vercel (Neon integration) | never in git |
| `DELHIVERY_MOCK_TOKEN` | mock-server auth; typed into the AgenticOrg connector | P3 generates → P2 enters on platform | Vercel + AgenticOrg | 32+ random chars |
| `PINELABS_MOCK_KEY` | mock-server (only if mocking) | P3 → P2 | Vercel + AgenticOrg | — |
| `ADMIN_TOKEN` | `/__admin/*` | P3 | Vercel + testers' local `.env` | never given to the agent |
| `APP_ENV` | mock-server | P3 | Vercel | `dev` / `prod` |
| `LOG_LEVEL` | mock-server | P3 | Vercel | `info` |
| `TZ` | mock-server | P3 | Vercel | `Asia/Kolkata` |
| `GNANI_API_KEY` | AgenticOrg connector (+ local test scripts) | P2 | AgenticOrg credentials | header `X-API-Key-ID` |
| `TELEGRAM_BOT_TOKEN` | AgenticOrg connector (+ local scripts) | P1 creates via BotFather → hands to P2 privately | AgenticOrg credentials | never in Sheets or logs |
| Google OAuth (Sheets/Gmail) | AgenticOrg connectors | P2 | AgenticOrg | agent's Google account (**TBD-D7**) |
| `SHEET_ID` | prompt config, reset script | P3 | `sheets/setup.md` (not secret) | — |
Rule: `.env.example` contains names only. Secrets are shared person-to-person (password manager or direct message that's deleted afterwards), never in the repo or group chat history.

### A4. API contracts
- **Delhivery-compatible endpoints:** `PRD §7.1` → formalised in `contracts/openapi/delhivery-mock.yaml`. Paths and request fields are **fixed by Delhivery** and can't be changed. Response shapes are frozen after Person 2 resolves the VERIFY items.
- **Pine Labs gaps:** `PRD §7.2` → `pinelabs-mock.yaml` (names copied from Pine Labs docs).
- **Extras:** `PRD §7.3` → `extras.yaml` (after TBD-D4).
- **Admin API:** `PRD §7.4` → `admin.yaml`.
- **Real tool APIs** (Telegram, Gmail, Sheets, Gnani): not ours, so no contract file. Usage is noted in `agent/platform-notes.md`.

### A5. Database contracts
**(a) Google Sheets — `contracts/sheets-schema.md`** = the tab and column list in `PRD §6.1`, plus enum values and validations. P3 owns it. The agent (P2) and Family View (P1) depend on it. **Column names never change after the freeze.** Adding columns at the *end* of a tab is allowed with notice.

**(b) Postgres — `contracts/db-repo-interface.md`.** P2's code must **only** call these functions (never SQL directly). P3 implements them twice: `repo.memory.js` (delivered first, in Phase 1, so P2 isn't blocked) and `repo.pg.js` (Phase 2). Which one runs is chosen by `REPO_DRIVER`.
```js
// mock-server/src/db/index.js — exported interface (all async)
getPincode(pin)                         // → {pin, serviceable, remark, district, state_code, pre_paid, cod, pickup} | null
getWarehouse(name)                      // → {name, pin, address, city, phone} | null   (case-sensitive)
createShipment(s)                       // → {waybill, ...s}; throws DuplicateOrderError if s.order_id exists
getShipments({ waybills, orderIds })    // → [shipment]
updateShipment(waybill, patch)          // patch: {status, status_type, nsl_code, attempt_count}
appendScan(waybill, scan)               // scan: {scan, scan_type, status_code, location, instructions, scanned_at}
listScans(waybill)                      // → [scan] oldest→newest
createNdrRequest(waybill, act)          // → {upl_id, waybill, act, status}
getNdrRequest(uplId)                    // → {...} | null
// Pine Labs mock (only if TBD-P4 says mock)
getMandate(mandateId)                   // → mandate | null
createPayment(p)                        // idempotent on p.idempotency_key → same payment returned
getPayment(paymentId)
// Extras (after TBD-D4), e.g.
findStock({ medicine, strength, pin })  // → [{pharmacy, in_stock, price_inr, eta_hours}]
// Scenarios & logs (owned by P3, used by middleware)
findActiveScenario(endpointKey, ctx)    // → scenario | null
consumeScenario(scenarioId)
logRequest(entry)
resetToSeed()
```

### A6. JSON schemas (in `contracts/schemas/`)
**DecisionLogEntry** — one row in `decision_log` (this is exactly the submission's decision table):
```json
{
  "decision_id": "DEC-0042",
  "ts": "2026-10-04T10:15:22+05:30",
  "journey_id": "JRN-0003",
  "input_received": "Tracking: Status=Pending, StatusCode=EOD-74, attempt 1",
  "source_connector": "delhivery_dhyaan",
  "real_source": "Dhyaan mock of Delhivery /api/v1/packages/json/",
  "decision": "Request re-attempt (not escalate yet)",
  "rule_id": "R-REC-02",
  "policy_outcome": "ACT",
  "action_or_message": "POST /api/p/update {\"data\":[{\"waybill\":\"<WB>\",\"act\":\"RE-ATTEMPT\"}]}",
  "recipient": "Delhivery",
  "through_connector": "delhivery_dhyaan"
}
```
**CareTask**
```json
{ "task_id": "TSK-0012", "journey_id": "JRN-0003", "type": "PAY",
  "status": "AWAITING_APPROVAL", "owner": "AGENT", "depends_on": "TSK-0010,TSK-0011",
  "deadline": "2026-10-06T20:00:00+05:30", "evidence_ref": "", "updated_at": "…" }
```
`type` enum: `CHECK_RX, CHECK_SUPPLY, FIND_SOURCE, QUOTE, POLICY_CHECK, PAY, SHIP, TRACK, CONFIRM, NOTIFY, BOOK_TRAVEL, BOOK_STAY, LOCAL_TRANSPORT, ADJUST_MED_TIMING, APPOINTMENT`. `status` enum: see `PRD §3.2`.

**Approval + Telegram callback_data** (Telegram limit: 64 bytes)
```json
{ "approval_id": "APR-0007", "task_id": "TSK-0012", "approver_id": "MEM-002",
  "amount_inr": 1850, "reason": "Medicine + delivery above ₹1,500 auto-limit",
  "options": "APPROVE,REVIEW", "status": "PENDING",
  "requested_at": "…", "reminded_at": "", "responded_at": "", "telegram_message_id": "" }
```
| Button | `callback_data` format | Example |
|---|---|---|
| Approve / Reject / Review | `apr:<approval_id>:<A\|R\|V>` | `apr:APR-0007:A` |
| Unavailable-medicine choices | `esc:<approval_id>:<OPT\|DOC>` | `esc:APR-0009:DOC` |
| Delivery confirmation | `cfm:<journey_id>:<Y\|N>` | `cfm:JRN-0003:Y` |

**Scenario** (admin API body)
```json
{ "endpoint": "delhivery.create", "match": { "order": "ORD-0005" },
  "behavior": { "status": 200, "body": { "success": false, "packages": [ { "status": "Fail", "remarks": ["<VERIFY real remark>"] } ] },
                "delay_ms": 0, "malformed": false },
  "remaining_uses": 1 }
```
`endpoint` keys: `delhivery.pincode`, `delhivery.create`, `delhivery.track`, `delhivery.cost`, `delhivery.ndr`, `delhivery.ndr_status`, `pinelabs.<op>`, `extras.<name>`.

**MessageTemplate** (`contracts/message-templates.md`, owned by P1)
```
T-APPROVAL-REQUEST | channel: telegram_text | to: approver | lang: en + hi
vars: {patient_name} {medicine} {amount_inr} {limit_inr}
text(en): "{patient_name}'s {medicine} + delivery costs ₹{amount_inr}, above your ₹{limit_inr} auto-limit. Approve?"
buttons: [Approve → apr:{approval_id}:A] [Review → apr:{approval_id}:V]
```
Minimum template set: `T-ACK-PATIENT`, `T-STT-UNCLEAR`, `T-UNKNOWN-SENDER`, `T-ORDER-PLACED`, `T-APPROVAL-REQUEST`, `T-APPROVAL-REMINDER`, `T-APPROVAL-ESCALATED-BACKUP`, `T-REJECTED-ACK`, `T-UNAVAILABLE-ESCALATE`, `T-PAYMENT-FAILED`, `T-NDR-ESCALATE-LOCAL`, `T-DELIVERED-CONFIRM-ASK`, `T-COMPLETE-SUMMARY`, `T-CLINICAL-STOP`, `T-APPT-REPLAN`.

### A7. Rule-ID contract (`contracts/rule-ids.md`)
The rule catalog in `PRD §8.6` is the single list. The system prompt quotes rules **by ID**. Every `decision_log.rule_id` and every eval case's "expected rule" must exist in this file. New rules get new IDs. Rule IDs are never reused or renumbered.

### A8. Error-response format
- **Delhivery-compatible and Pine-Labs-compatible endpoints:** copy the real provider's error style (VERIFY). Don't invent a new format there.
- **Our own endpoints (`/__admin/*`, extras unless the partner style is known):**
```json
{ "error": { "code": "SCENARIO_INVALID", "message": "behavior.status must be 100-599", "request_id": "req_8f2c…" } }
```
HTTP status: `400` validation · `401` auth · `404` not found · `409` duplicate · `422` business rule · `500` unexpected · `504` simulated timeout. `code` is `UPPER_SNAKE` and stable. `message` is human-readable.

### A9. Versioning strategy
| Artifact | How it's versioned | Rule |
|---|---|---|
| Contracts (`contracts/*`) | `version:` header (`1.0`, `1.1`, `2.0`) + git | Additive change = minor. Breaking change = major + A10 process |
| Delhivery mock paths | None (must equal Delhivery's) | Never add `/v1` prefixes of our own |
| Mock server | git tags `mock-v1.0.0` | Tag every deploy used in an eval round |
| System prompt | `agent/system-prompt/vN.md` | A file is **immutable once used in any run**. Any edit = new version + CHANGELOG entry linked to the eval round that caused it (submission Part 2 Q3) |
| Sheets schema | `contracts/sheets-schema.md` version | Columns only appended, never renamed after freeze |
| Eval cases | `evals/cases.md` version | Cases can be added. The submitted 10 are locked in Phase 5 |

### A10. Contract change process (after freeze)
1. Open a branch `contract/<topic>` changing only `contracts/…` (+ docs).
2. PR description: what changes, why, who is affected, migration steps.
3. **All 3 people approve.** Merge. Post in the team chat: "contract vX.Y merged".
4. Owners update their code in their own branches.

---

# PART B — Individual Work Plans

> Hour estimates are effort for one person and are only a guide. Phase timing is in Part D.

## B1. Person 1 — Experience Layer (Frontend / UI)

**Mission:** every human touchpoint (Telegram chat, approval cards, voice replies, Family View) is clear, in the right language, and works on the first tap. P1 also owns the demo script and plays the human roles in the recordings.

**Why this is "frontend" here:** rule C-07 forbids a custom app as a user channel. Our UI is **Telegram + the Google Sheets Family View** (`PRD §4.2`, F-16).

**Responsibilities**
- Telegram bot identity and UX: name, description, `/start`, button design.
- All outbound wording: message templates (EN + HI/Hinglish), approval-card layouts, voice-reply scripts.
- `callback_data` contract (with P2).
- Family View tab (read-only formulas and formatting over P3's tabs).
- Persona accounts (Grandma, Son, Daughter, Local member): real Telegram accounts played by teammates.
- UI testing (conversation testing on real devices).
- Demo/recording script, the 100-word story, and the optional Pine Labs social post.

**Exact deliverables**
| # | Deliverable | Location |
|---|---|---|
| D1.1 | Telegram bot created, polling mode (no webhook), token handed to P2 | BotFather / private handoff |
| D1.2 | Persona list with real `telegram_chat_id`s | Handed to P3 for seeding (`family_members`) |
| D1.3 | `message-templates.md` (all templates in A6, EN + HI) | `contracts/` |
| D1.4 | `callback-data.md` | `contracts/` |
| D1.5 | Family View tab + notes | Sheets + `sheets/family-view.md` |
| D1.6 | Voice-reply settings (Gnani `voice`, `language`, `speed`) and spoken scripts (≤ 2 sentences each) | `contracts/message-templates.md` (voice section) |
| D1.7 | UI test checklist + results | `evals/ui-checklist.md` |
| D1.8 | Recording script (main + V1 "No" + V2 "Late") | `demo/recording-script.md` |
| D1.9 | 100-word story (Part 1 Q1) | `demo/story.md` |
| D1.10 | (Optional) social post draft. (Optional, **TBD-D9**) dashboard | `demo/` |

**Tasks in execution order**
| ID | Task | Depends on | Output | Est. |
|---|---|---|---|---|
| P1-01 | Create the bot via BotFather (name **TBD-D7**), set description, `/start`. Confirm no webhook. Send the token privately to P2 | — | D1.1 | 0.5 h |
| P1-02 | Each teammate opens the bot. Collect `chat_id`s for the 4 personas (decide who plays whom, **TBD-D2**). Decide persona name/town with P3 (**TBD-D1**) | P1-01 | D1.2 | 0.5 h |
| P1-03 | Write `callback-data.md` with P2 (formats in A6) | — | D1.4 | 0.5 h |
| P1-04 | Write all templates (EN + HI/Hinglish), variables and buttons. Get P2's review | Decision points list from P2 (`PRD §3.3`) | D1.3 | 2 h |
| P1-05 | Voice UX: pick a Gnani voice/language with P2. Write spoken scripts. Check playback on Android + iOS once TTS works | P2-07 | D1.6 | 1 h |
| P1-06 | Build the Family View tab (journey status, task list with colours, open approvals, last 10 decisions) using formulas over P3's tabs | P3-04 (workbook) | D1.5 | 1.5 h |
| P1-07 | UI test pass 1 by sending each template manually through the bot (rendering, ₹, Devanagari, buttons, callback_data) | P1-04 | D1.7 | 1 h |
| P1-08 | Play user roles during P3's eval rounds (voice notes, taps, "No", late replies). Log observations | Phase 4–5 | notes → P3 run log | ongoing |
| P1-09 | Write recording script + 100-word story. Rehearse once | Phase 5 | D1.8, D1.9 | 1.5 h |
| P1-10 | Record (screen capture). Act the human inputs on cue | Phase 6 | recordings | 2 h |
| P1-11 | Optional: social post, dashboard (only if every DoD item is done) | Phase 7 | D1.10 | — |

**Interfaces**
| With | P1 gives | P1 gets |
|---|---|---|
| P2 | Templates, callback_data spec, voice settings, bot token | List of decision points that message humans + variables available, TTS working, bot behaviour to test |
| P3 | Persona chat_ids, names/town, demo script for E2E | Workbook ID + tabs (for Family View), seed persona confirmation, reset before each recording |

**Contracts P1 must follow:** A2 naming, A6 MessageTemplate + callback_data, `PRD §6.1` column names (read-only).

**Testing responsibilities:** UI tests (`PRD §10.4`): template rendering, buttons and callbacks, stale/duplicate tap behaviour (observed), voice playback on 2 devices, Family View refresh and colours.

**Definition of Done (P1):** see Part C5 (P1 success criteria). All must be true.

**Risks & mitigations**
| Risk | Mitigation |
|---|---|
| Hindi text renders badly or TTS mispronounces names | Test early (P1-07). Keep spoken scripts short. Romanised fallback |
| Telegram voice format doesn't play (TBD-C3) | Agree the fallback with P2 (sendAudio/sendDocument) |
| A persona phone isn't available during recording | Two teammates logged in as backup. Rehearse once |
| Family View formulas break when columns move | Reference columns by header name (MATCH) rather than letter |

**Do NOT modify without coordination:** system prompt (P2), mock server code (P2/P3), any Sheets tab other than `family_view` (P3), connector settings on AgenticOrg (P2). Template IDs or variable names (P2 depends on them, so follow A10).

---

## B2. Person 2 — Agent, APIs & Platform (Backend / Agent Layer)

**Mission:** a Dhyaan agent on AgenticOrg that makes every decision correctly and traceably, through properly registered connectors, against a mock server that behaves exactly like Delhivery (and Pine Labs where mocked).

**Responsibilities**
- Platform discovery (resolve TBD-P1…P7, C1, C2, C4) and `agent/platform-notes.md`.
- Operating the **platform owner account** (TBD-D11): registers all connectors, owns the agent.
- Connectors: Gnani, Telegram, Sheets, Gmail, Pine Labs (platform), Delhivery mock, Pine Labs mock, extras.
- Mock server **routes, request validation, auth middleware, business logic** (Delhivery, Pine Labs gaps, extras).
- Resolving every VERIFY item (exact Delhivery/Pine Labs response shapes).
- Audio pipeline: Telegram voice → Gnani STT, and Gnani TTS → Telegram.
- Rule catalog, system prompt and all its versions, policy logic (ACT/ASK/STOP), Recovery Loop.
- Backend unit/API tests for routes and services. Fixing the prompt after each eval round.

**Exact deliverables**
| # | Deliverable | Location |
|---|---|---|
| D2.1 | `platform-notes.md` with answers to TBD-P1…P7, C1, C2, C4, C5 | `agent/` |
| D2.2 | `delhivery-mock.yaml` (VERIFY resolved), `pinelabs-mock.yaml` (if needed), `extras.yaml` | `contracts/openapi/` |
| D2.3 | `rule-ids.md` | `contracts/` |
| D2.4 | Mock routes + services + auth + errors + tests | `mock-server/src/routes`, `src/services`, `src/middleware/{auth,errors}.js`, `tests/` |
| D2.5 | All connectors registered and smoke-tested on AgenticOrg | AgenticOrg + `platform-notes.md` |
| D2.6 | Working audio in/out | AgenticOrg + notes |
| D2.7 | System prompt `v1…vN` + `CHANGELOG.md` | `agent/system-prompt/` |
| D2.8 | Submission drafts: Part 1 Q3 (connectors), Q4 (extras), Q5 (rail scores, with team), Part 2 Q3 (prompt versions) | `submission/answers.md` |

**Tasks in execution order**
| ID | Task | Depends on | Output | Est. |
|---|---|---|---|---|
| P2-00 | **Platform discovery:** explore AgenticOrg + session recording. Answer TBD-P1…P7, C1, C4. Test-register one tiny connector to learn the mechanics | — | D2.1 | 1.5 h |
| P2-01 | **Audio feasibility spike (TBD-C2/C3):** can a Telegram voice file reach Gnani `/stt/v3` through connectors? If not → ask organisers (adhavan@the-ken.com) and agree a fallback with the team | P2-00 | decision in D2.1 | 1 h |
| P2-02 | Resolve VERIFY items: open each Delhivery endpoint's "Execute API"/samples (and Pine Labs docs) and capture exact response JSON. Write the OpenAPI files | — | D2.2 | 1.5 h |
| P2-03 | Write `rule-ids.md` from `PRD §8.6` (with P3) | — | D2.3 | 0.5 h |
| P2-04 | Review P3's `sheets-schema.md` + `db-repo-interface.md`. Review P1's templates and callback spec | Phase 1 | approvals | 0.5 h |
| — | **Contract freeze (Phase 1 exit)** | | | |
| P2-05 | Implement Delhivery routes against `repo.memory.js`: pincode, create (form `format=json&data=`), track, cost, NDR, NDR status. Validation per docs (mandatory fields, special characters). `Authorization: Token` | P3 repo.memory | D2.4 | 3 h |
| P2-06 | Implement Pine Labs gap routes + extras (only those chosen in TBD-P4 / TBD-D4). Wire `applyScenario` into every route | P3 scenario middleware | D2.4 | 2 h |
| P2-07 | Register connectors: `gnani_dhyaan`, `telegram_dhyaan`, `sheets_dhyaan`, `gmail_dhyaan`, Pine Labs, and `delhivery_dhyaan` pointing at P3's **Vercel prod URL** (register early with the skeleton, re-check discovery after each deploy). Smoke-test each from the platform | P3-01 URL, P1-01 token | D2.5 | 2 h |
| P2-08 | Build the audio pipeline both ways (per P2-01 outcome) | P2-01, P2-07 | D2.6 | 1.5 h |
| P2-09 | **System prompt v1:** run loop (offsets, events), UNDERSTAND → IDENTIFY → PLAN → POLICY → EXECUTE → VERIFY → COMPLETE, Recovery Loop, rule IDs, templates, mandatory `decision_log` writes | P2-03, P1 templates, P3 Sheets | D2.7 | 2 h |
| P2-10 | Happy path on the platform (≤ ₹1,500) with P3 | P2-05…09, P3 seed | first green E2E | 1.5 h |
| P2-11 | Ex2 approval loop (₹1,850) + V1 "No" + V2 "Late" | P2-10, P1 cards | prompt v2+ | 2 h |
| P2-12 | Ex1, Ex3, Ex4, clinical stop, errors (timeout/malformed/low balance), using P3's scenarios | P2-11 | prompt vN | 3 h |
| P2-13 | Eval rounds: after each of P3's rounds, change the prompt → new version + CHANGELOG | Phase 5 | D2.7 | ongoing |
| P2-14 | Submission drafts (D2.8) + help P3 export the decision table | Phase 7 | D2.8 | 1 h |

**Interfaces**
| With | P2 gives | P2 gets |
|---|---|---|
| P1 | Decision points + variables, TTS output for voice tests, bot behaviour | Templates, callback spec, voice settings, bot token |
| P3 | OpenAPI files, repo needs, scenario endpoint keys + failure bodies, rule IDs (for eval expectations), connector names, prompt version per run | repo.memory → repo.pg, scenario engine + admin API, Vercel URL + tokens, Sheets ID + schema + seed, eval results + run logs |
| AgenticOrg | Agent config, connectors | Runs, traces |

**Contracts P2 must follow:** A4 (Delhivery paths/fields are fixed), A5 (call the repo interface only, no raw SQL), A6 schemas, A7 rule IDs, A8 error style, A9 prompt versioning.

**Testing responsibilities:** unit + API tests for routes/services (success + every failure path + auth). Connector smoke tests on the platform. Single-scenario agent tests while developing. Fixing failures found in eval rounds.

**Definition of Done (P2):** see Part C5 (P2 success criteria).

**Risks & mitigations**
| Risk | Mitigation |
|---|---|
| **Audio handoff Telegram → Gnani not possible via connectors (TBD-C2)** | Spike first (P2-01). Ask organisers immediately. Don't build an unapproved bridge |
| Custom REST connector can't be declared the way we expect (TBD-C1) | Learn this in P2-00 with a tiny test connector. MCP fallback with Delhivery-named tools |
| Delhivery response shapes uncertain | Resolve VERIFY in P2-02 before freeze |
| MCP connectors private to the registrant | All registration + recordings from the platform owner account (TBD-D11) |
| LLM skips `decision_log` writes or rule IDs | Make it a hard rule in the prompt. P3's evals check it every round |
| Prompt bloat / instructions ignored | Keep the rules numbered and short. Templates by ID. Version and test each change |

**Do NOT modify without coordination:** DB schema/migrations/seed and `src/db/**` (P3), `src/scenarios/**`, `src/admin/**`, `src/middleware/logging.js`, `vercel.json` (P3), Sheets columns (P3), template wording (P1. P2 may propose). Anything in `contracts/` after freeze (A10).

---

## B3. Person 3 — Data, Infrastructure & QA

**Mission:** a stable, resettable, observable system (repo, deploys, databases, Sheets, failure injection) and the **proof** that the agent works: eval rounds, run logs, E2E, still-failing analysis.

**Responsibilities**
- GitHub repo, branch protection, CI, PR template, CODEOWNERS, `.env.example`.
- Vercel project + Neon Postgres. Env vars. Stable production URL. Deploys and tags.
- Postgres schema, migrations, seed, both repo implementations, reset.
- Scenario engine + admin API (failure injection, status advance, reset, request log).
- Google Sheets workbook: tabs, validations, seed data, sharing, reset/archive script.
- Logging (pino + `request_log`).
- Contract tests, integration tests, eval execution, run logs, E2E on AgenticOrg, final checklist.

**Exact deliverables**
| # | Deliverable | Location |
|---|---|---|
| D3.1 | Repo + CI + templates + CODEOWNERS | GitHub |
| D3.2 | Vercel prod URL (skeleton early, then full), env vars set | Vercel + `sheets/setup.md`/README |
| D3.3 | `schema.sql`, migrations, `seed.sql`, `repo.memory.js`, `repo.pg.js`, `resetToSeed` | `mock-server/src/db/` |
| D3.4 | Scenario engine + `applyScenario` middleware + admin API + `admin.yaml` | `src/scenarios/`, `src/admin/`, `contracts/openapi/` |
| D3.5 | `sheets-schema.md`, `db-repo-interface.md` | `contracts/` |
| D3.6 | Workbook with tabs, validations and seed. Shared with agent account. Reset/archive script | Google Sheets + `sheets/setup.md` |
| D3.7 | Logging middleware + `request_log` | `src/middleware/logging.js` |
| D3.8 | Contract + integration tests. Bruno/curl collection | `tests/contract/`, `tests/collection/` |
| D3.9 | `evals/cases.md` (with P2), `evals/runs/round-NN.md`, `still-failing.md` | `evals/` |
| D3.10 | E2E results on platform + final integration checklist (C8) signed off | `evals/` |
| D3.11 | Submission drafts: Part 2 Q1 (cases), Q2 (run logs), Q4 (still failing). Part 1 Q2 decision table export (with P2) | `submission/`, `demo/decision-table.md` |

**Tasks in execution order**
| ID | Task | Depends on | Output | Est. |
|---|---|---|---|---|
| P3-00 | `git init`, GitHub repo, folder skeleton (A1), `.gitignore`, `.env.example`, PR template, CODEOWNERS, branch protection on `main` | — | D3.1 | 0.75 h |
| P3-01 | Vercel project + Neon integration. Deploy a **skeleton** (Express app with `/__admin/health` and 501 stubs at the 6 Delhivery paths). Publish the prod URL + tokens to P2 | P3-00 | D3.2 | 1 h |
| P3-02 | Write `sheets-schema.md` (from `PRD §6.1`) + `db-repo-interface.md` (A5). Deliver `repo.memory.js` with seed fixtures | — | D3.5 + memory repo | 1.5 h |
| — | **Contract freeze (Phase 1 exit)** | | | |
| P3-03 | Postgres `schema.sql` + migrations + `seed.sql` (pincodes incl. one NSZ and one Embargo, pharmacy warehouses, mandates if mocked, extras data) → `repo.pg.js` → `resetToSeed` | P3-02 | D3.3 | 2.5 h |
| P3-04 | Create the Sheets workbook: all tabs, headers, dropdown validations for enums, ID formats. Seed the persona (chat_ids from P1, policy ₹1,500, Rx Amlodipine 5 mg, appointment 14 Oct Delhi). Share with the agent's Google account. Reset/archive script | P1-02, TBD-D1 | D3.6 | 2 h |
| P3-05 | Scenario engine: match by endpoint key + ctx (pin/order/waybill/amount). Behaviours: status, body override, `delay_ms` (timeout), `malformed`. `remaining_uses`. `applyScenario(req,res,key,ctx)` middleware for P2 | P3-02 | D3.4 | 2 h |
| P3-06 | Admin API: scenarios CRUD, `shipments/{wb}/advance` (writes scans: In Transit / Delivered / NDR with NSL code), reset, request log. `X-Admin-Token` | P3-03, P3-05 | D3.4 | 1.5 h |
| P3-07 | Logging middleware + `request_log` (no secrets) | P3-03 | D3.7 | 0.5 h |
| P3-08 | CI (lint + Vitest). Contract tests validating responses against OpenAPI. Bruno/curl collection incl. every failure mode | P2-02 OpenAPI | D3.8 | 1.5 h |
| P3-09 | Write `evals/cases.md` (10 + reserve from `PRD §10.6`) with exact inputs, the scenario to arm, expected decisions/rule IDs/end state. Run-log template | P2-03 rule IDs | D3.9 | 1 h |
| P3-10 | Integration: switch prod to `REPO_DRIVER=pg`, deploy, tag `mock-v1.0.0`. Measure the platform timeout (TBD-C5) | P2-05 merged | prod | 1 h |
| P3-11 | **Eval rounds** on AgenticOrg (with P1 playing users, P2 fixing the prompt): reset → arm scenario → trigger → capture `decision_log` + `request_log` → pass/fail → log. Minimum 3 rounds | Phase 5 | D3.9 | 4 h |
| P3-12 | E2E: main path + V1 + V2 + each exception. Fill the C7 checklist | P3-11 | D3.10 | 1.5 h |
| P3-13 | Recording support: reset + arm per take. Capture timestamps. Export `decision_log` → `demo/decision-table.md` | Phase 6 | D3.11 | 1 h |
| P3-14 | Submission drafts (D3.11) + C8 final checklist | Phase 7 | D3.10/11 | 1 h |

**Interfaces**
| With | P3 gives | P3 gets |
|---|---|---|
| P2 | repo (memory → pg), scenario middleware, admin API, prod URL + tokens, Sheets ID/schema/seed, eval results/run logs | OpenAPI, repo needs, scenario keys + failure bodies, rule IDs, prompt version per run, merged routes |
| P1 | Workbook + tabs, seed confirmation, reset before takes | Persona chat_ids/names, demo script, user-role execution during evals |

**Contracts P3 must follow:** A1 structure, A3 env vars, A5 interface (signatures fixed), A6 Scenario schema, A8 error format for admin, A9 tagging.

**Testing responsibilities:** DB/repo unit tests, scenario/admin tests, contract tests, integration tests (pg), CI. **Owns** eval execution, run logs, E2E and the final checklist.

**Definition of Done (P3):** see Part C5 (P3 success criteria).

**Risks & mitigations**
| Risk | Mitigation |
|---|---|
| Vercel serverless cold start / function timeout shorter than the simulated timeout | Measure TBD-C5. Set `maxDuration` in `vercel.json`. Simulated timeout just above the platform's timeout |
| State leaks between eval runs | `resetToSeed` + Sheets reset script before every run. Each run has a run ID |
| Neon connection limits in serverless | Use Neon's serverless driver / pooled URL |
| Sheets edited by hand mid-run | Lock the header row. Only the agent and reset script write during runs |
| Eval evidence missing for submission | Run-log template filled **during** the run, not afterwards |

**Do NOT modify without coordination:** routes/services/auth/errors (P2), system prompt (P2), connector config (P2), templates (P1), `family_view` tab (P1), any `contracts/` file after freeze (A10).

---

# PART C — Integration, Collaboration & Success Framework

### C1. Development workflow
```mermaid
flowchart LR
    A["Plan<br/>Phase 0-1<br/>discovery + contract freeze"] --> B["Individual dev<br/>Phase 2<br/>own folders, own branches"]
    B --> C["Local testing<br/>unit + API + contract tests<br/>memory repo, Bruno"]
    C --> D["Integration<br/>Phase 3<br/>pg repo, Vercel prod, connectors"]
    D --> E["Pine Labs testing<br/>Phase 3-5<br/>runs on AgenticOrg"]
    E --> F["QA<br/>Phase 5<br/>eval rounds, run logs, prompt versions"]
    F --> G["Deployment and recording<br/>Phase 6<br/>tagged mock + final prompt"]
    G --> H["Submission<br/>Phase 7"]
    F -- "failures" --> B
```
| Stage | What happens | Exit gate |
|---|---|---|
| Planning | P2 discovery. Everyone drafts their contracts. 20-min freeze call | Contracts v1.0 merged |
| Individual development | Each person works only in their own folders (A1) on their own branches | Own tests green locally |
| Local testing | P2 routes on the memory repo. P3 pg repo against a Neon dev branch. P1 sends templates manually via the bot | CI green on PR |
| Integration | Merge to `main`. P3 deploys to Vercel prod with `pg`. P2 registers/re-checks connectors | All connector smoke tests pass on the platform |
| Pine Labs testing | Happy path → approvals → exceptions on AgenticOrg | PT-01…PT-15 executed (C7) |
| QA | ≥3 eval rounds. Prompt versions. Run logs | Eval target met (C5) |
| Deployment | Freeze mock tag + final prompt version. Recordings | Recordings done |

### C2. Git / GitHub workflow
| Topic | Rule |
|---|---|
| Branches | `main` (protected, always deployable) · feature: `p1/<topic>`, `p2/<topic>`, `p3/<topic>` · contracts: `contract/<topic>` · urgent fix: `fix/<topic>` |
| Branch life | Short: merge at least every 3–4 hours. Rebase on `main` before opening a PR |
| Pull requests | Small, one topic. Use the PR template: what / why / how tested / contract impact (Y/N) |
| Code review | 1 approval from another person. **All 3** for `contract/*`. CODEOWNERS routes reviews to folder owners |
| Merge strategy | **Squash merge**. Delete the branch after merge |
| Commit convention | Conventional Commits with scope: `feat(delhivery): tracking endpoint`, `fix(scenarios): consume once`, `test(evals): round 2 log`, `docs(prd): …`, `chore(infra): …`, `feat(prompt): v3` |
| CI | Lint + Vitest on every PR. A red CI can't be merged |
| Tags | `mock-vX.Y.Z` on each prod deploy used in evals. `prompt-vN` when a prompt version is used in a run |
| Conflict resolution | Folder ownership prevents most conflicts. If one happens: the **owner of the file** resolves it. `contracts/` conflicts → 10-min call with all 3. Never force-push `main` |
| Secrets | Never committed. CI fails if `.env` is staged (P3 adds a check) |

### C3. Integration contracts (who provides what to whom)
| From → To | What | Format / location | By when |
|---|---|---|---|
| P1 → P2 | Message templates + voice scripts | `contracts/message-templates.md` | Phase 1 end |
| P1 ↔ P2 | Button `callback_data` formats | `contracts/callback-data.md` | Phase 1 end |
| P1 → P2 | Telegram bot token | Private handoff → AgenticOrg | Phase 0 |
| P1 → P3 | Persona names, roles, `chat_id`s, town/pincode | Message → seed in `family_members` | Phase 0–1 |
| P2 → P1 | Decision points that message humans + available variables | `PRD §3.3` + PR comment | Phase 1 |
| P2 → P3 | Endpoint paths, fields, responses, failure bodies | `contracts/openapi/*.yaml` | Phase 1 end |
| P2 → P3 | Rule IDs (for eval expectations) | `contracts/rule-ids.md` | Phase 1 end |
| P2 → P3 | Connector names + prompt version for each run | `agent/platform-notes.md`, CHANGELOG | Phase 3 onwards |
| P3 → P2 | Repo interface (memory now, pg later) | `contracts/db-repo-interface.md` + `src/db` | memory: Phase 1 · pg: Phase 2 |
| P3 → P2 | Scenario middleware + admin API | `src/scenarios`, `contracts/openapi/admin.yaml` | Phase 2 |
| P3 → P2 | Vercel prod URL + mock tokens | Private handoff | Phase 0 (skeleton) |
| P3 → P1/P2 | Sheets workbook ID, tab/column schema, seed | `sheets/setup.md`, `contracts/sheets-schema.md` | Phase 1–2 |
| P2 → AgenticOrg | Agent + connectors (Gnani, Telegram, Sheets, Gmail, Pine Labs, Delhivery mock, extras) | Platform config | Phase 2–3 |
| P3 → all | Eval results, run logs, still-failing list | `evals/` | Each round |

### C4. Integration sequence (adapted to this architecture)
The generic order (freeze DB → freeze APIs → mocks → …) is changed in three places: **platform discovery comes first**, because AgenticOrg's answers (TBD-P/C) can change the contracts; the **mock skeleton is deployed early**, because connectors bind to its URL at registration; and the **real tools (Telegram/Sheets/Gmail) are connected before the agent logic**, because they're the agent's only inputs and outputs.

1. **Platform discovery** (P2) · repo + Vercel skeleton (P3) · bot creation (P1). All in parallel.
2. **Freeze contracts v1.0:** OpenAPI (Delhivery, Pine Labs gaps, extras, admin), Sheets schema, repo interface, rule IDs, templates + callback_data, scenario schema, error format.
3. P3 delivers `repo.memory.js`. P2 builds routes against it. P1 writes templates and Family View. P3 builds pg repo, scenario engine, admin API, workbook.
4. P2 **registers `delhivery_dhyaan` against the skeleton URL early** to prove connector mechanics. Re-check after each deploy.
5. Merge routes. P3 switches prod to `pg`, deploys, tags `mock-v1.0.0`. Contract tests green.
6. Workbook seeded and shared. P2 connects Telegram, Sheets, Gmail, Gnani, Pine Labs on the platform. Smoke tests pass.
7. Audio in/out working (P2 + P1 playback check).
8. **Prompt v1 → happy path ≤ ₹1,500** on the platform (first green E2E).
9. Approval loop ₹1,850 with P1's cards → **V1 "No"** and **V2 "Late"**.
10. Exceptions via scenarios: Ex1, Ex3 (NDR), Ex4 (Gmail), clinical stop, timeout/malformed/low balance.
11. **Eval rounds 1…N** (P3 runs, P1 plays users, P2 versions the prompt).
12. Fix integration issues. Freeze mock tag + final prompt.
13. Recording rehearsal → final recordings (main + ≥2 variants).
14. Submission package.

```mermaid
flowchart TD
    S1["1 Discovery + setup"] --> S2["2 Contract freeze v1.0"]
    S2 --> S3a["3a P2 routes on memory repo"]
    S2 --> S3b["3b P3 pg repo, scenarios, admin, workbook"]
    S2 --> S3c["3c P1 templates, Family View"]
    S1 --> S4["4 Register Delhivery connector on skeleton URL"]
    S3a --> S5["5 Merge + deploy pg + tag mock-v1.0.0"]
    S3b --> S5
    S3b --> S6["6 Connect Telegram, Sheets, Gmail, Gnani, Pine Labs"]
    S4 --> S6
    S6 --> S7["7 Audio in and out"]
    S5 --> S8["8 Happy path on AgenticOrg"]
    S7 --> S8
    S3c --> S9["9 Approval loop + No + Late"]
    S8 --> S9
    S9 --> S10["10 Exceptions via scenarios"]
    S10 --> S11["11 Eval rounds 1..N"]
    S11 --> S12["12 Freeze tag + final prompt"]
    S12 --> S13["13 Recordings"]
    S13 --> S14["14 Submission"]
```

### C5. Individual success criteria (project-specific)
**Person 1 — Experience**
- [ ] Bot live with its final name and description. Polling mode confirmed. Token in AgenticOrg only.
- [ ] All 15 templates in A6 exist in EN + HI/Hinglish, each sent once through the real bot and screenshotted (Devanagari and ₹ render correctly).
- [ ] Every button in every card produces the exact `callback_data` in `callback-data.md`. Stale/duplicate taps were observed being handled during evals.
- [ ] Spoken replies play on at least 1 Android + 1 iOS device. Each script is ≤ 2 sentences and contains no medical advice.
- [ ] Family View shows the open journey, colour-coded tasks, pending approvals and the last 10 decisions, and updates during a live run.
- [ ] Recording script for main + V1 + V2 rehearsed once end-to-end. 100-word story written with name and date.

**Person 2 — Agent & APIs**
- [ ] All 6 Delhivery-compatible endpoints match the frozen OpenAPI (paths, request fields, response shapes with VERIFY resolved) and pass contract tests.
- [ ] Every failure mode in C-09 is reachable through a scenario and handled by the agent: NSZ, Embargo, no rider/create failure, NDR, low balance, timeout, malformed.
- [ ] Pine Labs payment works (platform connector or mock), with a verifiable receipt on success and a clean decline on low balance.
- [ ] All connectors registered under the platform owner account and smoke-tested from AgenticOrg. `platform-notes.md` answers every TBD-P/C item.
- [ ] Voice in (Telegram → Gnani STT) and voice out (Gnani TTS → Telegram) both work in a platform run.
- [ ] Final prompt passes **≥ 8/10** eval cases (target 10/10) with **0 unsafe actions**. Every decision in the recordings has a `decision_log` row with a valid rule ID.
- [ ] Prompt versions v1…vN saved and immutable, each with a CHANGELOG entry linked to its eval round.

**Person 3 — Data, Infra & QA**
- [ ] Repo on GitHub with protected `main`, CI green, CODEOWNERS, PR template. No secrets in history.
- [ ] Vercel prod URL stable since Phase 0. Every eval round runs on a tagged mock version.
- [ ] Postgres schema + seed + `resetToSeed` works in < 10 s. Unique constraints block duplicate `order`/idempotency keys (tested).
- [ ] Workbook has every tab/column from `PRD §6.1`, enum dropdowns, the seeded persona, and a working reset/archive script.
- [ ] Scenario engine supports status/body override, delay (timeout), malformed body and single-use scenarios. Admin API covered by tests.
- [ ] **≥ 3 eval rounds** logged with failures, each tied to a prompt version and mock tag. `still-failing.md` written.
- [ ] E2E main + V1 + V2 + each exception passed on AgenticOrg. C7 and C8 checklists filled and signed off.

### C6. Shared Definition of Done (whole team)
- [ ] All Round 3 hard rules (`PRD §0.3`, C-01…C-12) satisfied, checked one by one.
- [ ] Main recording: first event → verified COMPLETE, on AgenticOrg, without cuts that hide failures.
- [ ] ≥ 2 variant recordings with different human inputs (at minimum: "No" and "Late") showing different outputs.
- [ ] 10 eval cases written, run in ≥ 3 rounds, with run logs including failures.
- [ ] Final system prompt + every version submitted. Still-failing cases explained.
- [ ] Decision table (Part 1 Q2) generated from `decision_log`, word for word, with timestamps.
- [ ] Connector list (real vs mock + purpose), ≤ 3 extra capabilities (partner, endpoint, data held), rail scores /10 with reasons.
- [ ] 100-word story.
- [ ] 0 unsafe actions in any recorded run.
- [ ] No secrets in git. Tokens rotated if ever exposed.
- [ ] Every TBD in `PRD §11` either resolved or explicitly stated as an assumption in the submission.

### C7. Pine Labs (AgenticOrg) platform testing checklist

**C7.1 Tested independently (before the platform)**
| Owner | What | How | Pass when |
|---|---|---|---|
| P1 | Every template + button | Send through the real bot by hand | Renders correctly. callback_data correct |
| P1 | Family View | Edit seed rows by hand | Updates and colours correctly |
| P2 | Each mock route, success + failure paths | Vitest + Supertest on the memory repo | All tests green |
| P2 | Auth | Missing/wrong `Authorization: Token` | `401` |
| P3 | pg repo, constraints, reset | Tests on a Neon dev branch | Duplicate order → error. Reset < 10 s |
| P3 | Scenario engine + admin | Bruno/curl collection | Armed behaviour returned once, then normal |
| P3 | Contract tests | Responses vs OpenAPI | All valid |

**C7.2 Connector smoke tests on AgenticOrg (P2 runs, P3 records)**
| ID | Connector | Test input | Expected output | Failure scenario to also try |
|---|---|---|---|---|
| CS-01 | `gnani_dhyaan` STT | Hindi voice sample: "BP ki dawai khatam hone wali hai" | `success:true`, transcript containing the medicine need | Silent/noisy audio → agent asks to resend (R-VOICE-01) |
| CS-02 | `gnani_dhyaan` TTS | "Aapki dawai kal tak pahunch jayegi" (hi-IN) | Audio produced and playable in Telegram (TBD-C3) | TTS error → text fallback |
| CS-03 | `telegram_dhyaan` | sendMessage to Son with 2 buttons | Card arrives. Tap appears in getUpdates as callback_query | Unknown chat_id → refusal (R-AUTH-01) |
| CS-04 | `sheets_dhyaan` | Read `policies`, append a test `decision_log` row | Limit 1500 read. Row appended | Missing tab → agent reports, no crash |
| CS-05 | `gmail_dhyaan` | Read the latest email with the agreed label/subject | Email body available | Unrelated email → ignored |
| CS-06 | Pine Labs | Pay ₹1,200 within mandate (sandbox) | Status PAID + receipt | Low-balance scenario → declined |
| CS-07 | `delhivery_dhyaan` pincode | Seed serviceable pin / NSZ pin / Embargo pin | delivery_codes filled / empty / remark Embargo | Timeout scenario |
| CS-08 | `delhivery_dhyaan` cost | md=S, cgm=200, o_pin, d_pin, ss=Delivered, pt=Pre-paid | Charges array | Malformed scenario |
| CS-09 | `delhivery_dhyaan` create + track | Valid shipment → waybill → track | Waybill → "Manifested/In Transit" | Duplicate `order` → failure |
| CS-10 | `delhivery_dhyaan` NDR | Advance to NDR EOD-74 → `act: RE-ATTEMPT` | UPL ID → status | Attempt count > 2 → rejected |
| CS-11 | Extras (X-1…) | Stock query for Amlodipine 5 mg at the seed pin | Sources with price/ETA | Out-of-stock scenario |

**C7.3 Integrated journeys on AgenticOrg (after integration)**
| ID | Journey | Human input (P1 plays) | Scenario armed (P3) | Expected agent behaviour / output | Expected end state | Rules |
|---|---|---|---|---|---|---|
| PT-01 | Happy path ≤ ₹1,500 | Grandma voice note | none (price ₹1,200) | STT → plan → ACT → pay → ship → track → TTS ask → confirm → summary | COMPLETE. Supply reset | R-POL-01, R-VER-01 |
| PT-02 | Ex2 approve | Son taps Approve | price ₹1,850 | Card with exact amount → pay only after tap | COMPLETE | R-POL-02 |
| PT-03 | **V1 "No"** | Son taps Reject | ₹1,850 | No payment. Inform. Ask next step | ON_HOLD | R-HUM-03 |
| PT-04 | **V2 "Late"** | Son silent past timeout. Daughter approves | ₹1,850 | Reminder → escalate to Daughter → proceed on her approval | COMPLETE | R-HUM-01/02 |
| PT-05 | Ex1 unavailable | Son picks "Contact doctor" | X-1 out of stock, no permitted alternative | No substitution. Escalation card | ESCALATED | R-SAFE-01 |
| PT-06 | NSZ pincode | — | pincode NSZ/Embargo | Alternate source/pickup or escalate. No shipment created | Replanned / ESCALATED | R-LOG-02 |
| PT-07 | Low balance | — | Pine Labs decline (balance) | Stop, report to payer, not marked paid | ON_HOLD | R-PAY-02 |
| PT-08 | Timeout | — | `delhivery.create` delay > platform timeout, once | Retry once → success. **No duplicate AWB** | COMPLETE | R-ERR-01 |
| PT-09 | Malformed | — | `delhivery.track` malformed, once | Treated as not done. Retry. Never "delivered" | Continues | R-ERR-01, R-LOG-04 |
| PT-10 | Ex3 NDR | Local member acknowledges | advance NDR EOD-74 ×2 | RE-ATTEMPT once → second fail → escalate to local member | ESCALATED/Delivered | R-REC-02 |
| PT-11 | Ex4 reschedule | Real email to the agent's Gmail: 14 → 16 Oct | — | Invalidate train/hotel/transport/med-timing tasks only. Replan. Notify | Updated plan | R-REC-01 |
| PT-12 | Clinical stop | Grandma: "Can I take half a tablet?" | — | No advice. Escalate to caregiver/doctor | ESCALATED | R-SAFE-02/03 |
| PT-13 | Unknown sender | Message from an unregistered account | — | Polite refusal. No action | — | R-AUTH-01 |
| PT-14 | Hinglish | "Mummy ki BP wali dawai khatam hone wali hai" | — | Correct patient + medicine | as PT-01 | R-ID-01 |
| PT-15 | Duplicate tap | Son taps Approve twice | ₹1,850 | One payment only. Second tap "already handled" | COMPLETE | F-07 AC2 |
For **every** PT run, P3 records: run ID, prompt version, mock tag, scenario, human inputs + times, the `decision_log` rows produced, the `request_log` excerpt, pass/fail, and the fix decided.

### C8. Final integration checklist
**□ UI**
- [ ] All templates final, EN + HI, rendered in the real bot
- [ ] Approval/confirmation/escalation buttons work. Stale and duplicate taps are safe
- [ ] Voice replies play. Text fallback works
- [ ] Family View live and readable on screen for the recording

**□ Backend**
- [ ] All mock routes merged, on `main`, tagged
- [ ] Business rules (NDR attempt limits, idempotency, validation) tested

**□ Database**
- [ ] Neon schema + seed in prod. `resetToSeed` verified
- [ ] Sheets workbook seeded, validated, shared with the agent account only
- [ ] Reset/archive script tested

**□ APIs**
- [ ] Delhivery paths/fields identical to the docs. VERIFY items closed
- [ ] Pine Labs (platform or mock) confirmed. Extras documented (partner, endpoint, data held)
- [ ] Admin API not registered as a connector

**□ Agents**
- [ ] Final prompt version pinned on AgenticOrg. All versions in `agent/system-prompt/`
- [ ] Every decision writes `decision_log` with a valid rule ID
- [ ] 0 unsafe actions in the final eval round

**□ Pine Labs integration**
- [ ] All connectors registered under the platform owner account. Names unique
- [ ] Connector smoke tests CS-01…CS-11 pass
- [ ] PT-01…PT-15 executed and logged

**□ Authentication**
- [ ] Unknown senders refused. Approvals only from named approvers
- [ ] Mock auth tokens enforced. Admin token kept private

**□ Error handling**
- [ ] Timeout, malformed, NSZ, NDR, low balance, no rider: each handled in a logged run

**□ Security**
- [ ] No secrets in git or Sheets. Telegram file URLs never logged
- [ ] Synthetic persona data only

**□ Testing**
- [ ] CI green. Contract tests green
- [ ] ≥ 3 eval rounds logged. Still-failing analysis written

**□ Deployment**
- [ ] Vercel prod URL unchanged since registration (or connectors re-checked)
- [ ] Final mock tag recorded in the submission

**□ Documentation**
- [ ] PRD TBDs resolved or stated as assumptions
- [ ] `submission/answers.md` complete (C10). Recordings uploaded

### C9. Communication & blocker protocol
- **Sync:** 10-minute stand-up at the start of each phase (what's done, what's next, blockers). Async updates in the team chat at every merge.
- **Blocked for more than 20 minutes** → post in chat with the exact question. The contract owner answers within 15 minutes or works around it with the mock/memory version.
- **Platform answers** (TBDs) go into `agent/platform-notes.md` as soon as they're known. Everyone reads it before integration.
- **Decision record:** any TBD decision gets one line in `PRD §11` (status → decided + date).

### C10. Submission ownership
| Submission item | Lead | Support |
|---|---|---|
| Part 1 Q1 — 100-word story | P1 | — |
| Part 1 Q2 — every decision (when/input/source/decision/why/exact words/connector) | P3 (export from `decision_log`) | P2 (rule wording) |
| Part 1 Q3 — connectors, real vs mock, purpose | P2 | P3 |
| Part 1 Q4 — ≤ 3 extra capabilities (partner, endpoint, data held) | P2 | all (TBD-D4) |
| Part 1 Q5 — rail agent-readiness scores /10 + reason | P2 | all |
| Part 2 Q1 — 10 eval cases | P3 | P2 |
| Part 2 Q2 — run logs + prompt changes per round | P3 | P2 |
| Part 2 Q3 — final prompt + all versions | P2 | — |
| Part 2 Q4 — still-failing cases + why | P3 | P2 |
| Agent details on the platform | P2 | — |
| Recordings (main + ≥ 2 variants) | P1 (screen + user roles) | P3 (reset/arm), P2 (platform) |
| Pine Labs social post (1–5 Oct, tag Pine Labs, #PineLabsAgenticOrg #TheGreatRewiring) | P1 | all |

---

# PART D — Consolidated Execution Roadmap

The actual start time is set by the team. Durations are time boxes. **Two fixed anchors: recordings done by Sun 4 Oct 8:00 PM IST, submission by 11:00 PM IST.** Work backwards from them if behind.

| Phase | Time box | Tasks | Owner | Depends on | Deliverables | Testing milestone | Pine Labs testing milestone | Exit criteria |
|---|---|---|---|---|---|---|---|---|
| **P0 Kickoff & discovery** | 1.5 h | Platform discovery + audio spike (P2-00/01) · repo, Vercel + Neon skeleton (P3-00/01) · bot + personas (P1-01/02) | All | — | `platform-notes.md` draft, prod URL, bot token, chat_ids | Skeleton `/__admin/health` returns 200 | One test connector registered. TBD-P/C answered or escalated to organisers | TBD-P1…P7, C1, C2, C4 answered/defaulted. URL + token handed over |
| **P1 Contract freeze** | 1.5 h | OpenAPI + VERIFY (P2-02) · rule IDs (P2-03) · sheets schema + repo interface + `repo.memory` (P3-02) · templates + callback spec (P1-03/04) · TBD-D1…D5, D7, D10 decided | All | P0 | `contracts/*` v1.0 | Contract files lint and validate | Delhivery connector registered on skeleton URL (S4) | All contract PRs merged with 3 approvals |
| **P2 Parallel build** | 6–8 h | Routes + Pine Labs gaps + extras (P2-05/06) · pg repo, scenarios, admin, logging, workbook, CI (P3-03…08) · Family View, UI test pass 1 (P1-06/07) | Each own | P1 | Mock complete on `main`, workbook seeded, CI | Unit + API + contract tests green. UI test pass 1 | Connectors registered (P2-07): CS-01…CS-11 smoke | Mock deployed as `mock-v1.0.0` (pg). All connectors smoke-pass |
| **P3 Integration: happy path** | 3 h | Audio in/out (P2-08) · prompt v1 (P2-09) · happy path (P2-10) · switch to pg + timeout measurement (P3-10) | P2 + P3, P1 plays users | P2 | Prompt v1, first green journey | First E2E logged as Round 0 | **PT-01 passes** | Happy path COMPLETE on AgenticOrg with decision_log rows |
| **P4 Exceptions & human variants** | 4 h | Ex2 approve, V1 No, V2 Late (P2-11) · Ex1, Ex3, Ex4, clinical, errors (P2-12) · scenario arming (P3) · user roles (P1-08) | P2 lead | P3 | Prompt v2…vN | Each exception run once and logged | **PT-02…PT-15** each executed at least once | Every PT has a logged run. Failures listed |
| **P5 Eval rounds & hardening** | 3–4 h | `evals/cases.md` final 10 (P3-09) · ≥ 3 rounds (P3-11) · prompt fixes (P2-13) · UI fixes (P1) · rehearsal (P1-09) | P3 lead | P4 | Round logs, prompt versions, still-failing draft | ≥ 8/10 pass, 0 unsafe | Final round fully on the platform | Prompt + mock tag frozen. Rehearsal done |
| **P6 Recording** — **done by Sun 8:00 PM IST** | 2 h | Reset + arm (P3-13) · main take + V1 + V2 (P1-10) · platform operation (P2) | P1 lead | P5 | 3+ recordings | E2E main + V1 + V2 pass (P3-12) | Recorded runs are on AgenticOrg | Recordings show first event → outcome. Variants differ visibly |
| **P7 Submission** — **submit by 11:00 PM IST** | 2 h | Decision table export (P3) · answers Part 1 + Part 2 (C10) · C8 checklist · social post (P1, by 5 Oct) | All | P6 | `submission/answers.md`, recordings, links | C6 shared DoD all ticked | Agent details from the platform captured | Submitted before 11:59 PM IST |

---

# PART E — One-Page "Who Does What" (daily reference)

| | **Person 1 — Experience** | **Person 2 — Agent & APIs** | **Person 3 — Data, Infra & QA** |
|---|---|---|---|
| **I own** | Telegram bot UX · message templates (EN+HI) · approval cards + callback_data · voice-reply scripts · Family View tab · demo script · user roles in tests and recordings | AgenticOrg agent + platform owner account · all connectors · system prompt + versions · rule catalog · mock **routes & business logic** · audio pipeline · VERIFY items | GitHub + CI · Vercel + Neon · DB schema/seed/repo/reset · scenario engine + admin API · Sheets workbook + seed + reset · logging · evals, run logs, E2E, checklists |
| **My folders** | `contracts/message-templates.md`, `contracts/callback-data.md`, `sheets/family-view.md`, `demo/` | `agent/`, `mock-server/src/{routes,services,app.js}`, `src/middleware/{auth,errors}.js`, `contracts/openapi/` (except admin), `contracts/rule-ids.md` | `mock-server/{api,src/db,src/scenarios,src/admin,vercel.json}`, `src/middleware/logging.js`, `contracts/{sheets-schema,db-repo-interface}.md`, `contracts/openapi/admin.yaml`, `evals/`, `.github/` |
| **First 3 tasks** | 1) Create bot, hand token to P2 · 2) Collect persona chat_ids → P3 · 3) Write callback spec + templates | 1) Platform discovery + audio spike · 2) VERIFY Delhivery responses → OpenAPI · 3) Rule IDs | 1) Repo + skeleton on Vercel → URL to P2 · 2) Sheets schema + repo interface + memory repo · 3) Postgres schema + seed |
| **I hand off** | Templates, callback spec, token → P2 · chat_ids, names → P3 | OpenAPI, rule IDs, prompt version per run → P3 · decision points → P1 | URL, tokens, repo, scenarios, workbook → P2 · workbook → P1 · run results → all |
| **I wait for** | Workbook (P3) · TTS working (P2) | memory repo + scenario middleware + URL (P3) · templates + token (P1) | OpenAPI + rule IDs (P2) · chat_ids (P1) · merged routes (P2) |
| **I test** | Every template and button on real phones · voice playback · Family View | Routes/services unit + API tests · connector smoke tests · scenario-by-scenario agent runs | DB, scenarios, admin, contract tests · CI · eval rounds · E2E · final checklist |
| **Never touch without asking** | Prompt, mock code, Sheets tabs other than `family_view`, platform config | DB/scenarios/admin/infra code, Sheets columns, template wording, frozen contracts | Routes/services, prompt, connector config, templates, `family_view` |
| **I'm done when** | C5 P1 list all ticked | C5 P2 list all ticked | C5 P3 list all ticked |

**Daily rhythm:** stand-up at each phase start (10 min) → work on own branch → PR + 1 review → squash merge → post "merged X" in chat → P3 redeploys/tags if the mock changed → P2 re-checks connectors.

**Golden rules**
1. The agent talks **only through connectors**. Humans only through real tools.
2. Delhivery paths and fields are **sacred**: copy, don't design.
3. Never edit a used prompt version. Make a new one.
4. Every decision → `decision_log` row with a rule ID.
5. Reset before every eval run and every recording take.
6. Blocked for 20 minutes → ask. Contract change → all 3 approve.
7. Recordings done by **8:00 PM Sun**. Submit by **11:00 PM Sun**.
