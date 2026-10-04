# SaaS Launchup — current-state audit and sequential Codex build plan

**Prepared:** October 4, 2026  
**Repository:** https://github.com/SharpConceptions/SaasLaunchUp  
**Audited main commit:** `13940c30651e81d682ca181f4975d232245f37ba` — October 4, 2026, “local testing”  
**Observed live pages:** `https://www.saaslaunchup.com/` and `https://www.saaslaunchup.com/launch`  
**Recommended repository filename:** `docs/SAASLAUNCHUP_EXECUTION_PLAN.md`

## 1. Read this first

SaaS Launchup already has a substantial CRM foundation and newer code for native authentication, onboarding, Twilio calling/SMS, public token-based demo booking, and provider connections. Preserve these implementations and finish their missing behavior. The largest remaining product gap is the evidence layer connecting acquisition, trial activity, payment, customer experience, and retention to useful recommendations and measured improvements.

This is a **current source audit plus a limited live-site inspection**, not a certification of every production workflow. The public homepage and sign-in screen were inspected. The private workspace required an approved account; no production login, customer messages, calls, purchases, campaign changes, or database changes were performed. The deployed commit and live provider readiness remain unverified. Task T00 closes that gap.

The local September 26 audit describes an older branch state. Current main differs materially: the old `/api/appointments` implementation, its tests, and earlier product/status documents are absent from the current tree. Current booking uses `/api/demo-bookings` and `demo_appointments`. **Do not apply the old task list blindly or restore old migrations over the current database.** Current `0016` is a different migration from the older booking branch.

### How to use this document

1. Add this file to the current repository at the recommended path.
2. Give Codex the starter prompt in section 8 and ask for **T00 only**.
3. After reviewing that result, say: `Execute T01 only using the execution plan.` Continue with one task ID at a time.
4. For each task, Codex cross-references current code first. If work is already complete, it proves the acceptance criteria and records it as verified; it does not rebuild it.
5. Use one focused branch/PR per task. Use CodeRabbit to review that diff, then resolve relevant findings and record test evidence. Review comments are evidence to evaluate, not automatic proof that the product works.
6. Do not ask an agent to implement the whole document in a single change. Later tasks are the full roadmap; they are not all prerequisites for a useful beta.

## 2. What was verified now

The current GitHub tree was retrieved through the connected repository integration. A local source snapshot was reconstructed for inspection. Its 309 downloaded/reused text files were checked against Git blob hashes after normalizing line endings and final newlines. Images, the vendored minified voice SDK, generated TypeScript build metadata, and historical Drizzle schema snapshots were omitted from this audit copy; application source, SQL migrations, the migration journal, and lockfile were included. Local compilation does not validate those omitted static assets or the production deployment.

| Check | October 4 result | What it proves / does not prove |
| --- | --- | --- |
| Public homepage | Loads; redirects to `www.saaslaunchup.com`; approved-account notice | Public entry works in the inspected browser. It does not prove authenticated workflows. |
| Sign-in page | Email/password, Google, recovery and setup links visible | Sign-in UI exists. No credentials were submitted. Google button text appeared low contrast and needs a measured accessibility check. |
| `npm ci --ignore-scripts --no-audit --no-fund` | Passed; 679 packages installed | Lockfile installs in this environment with install scripts disabled. |
| `npm run build` | Exit 0; server bundle and Wrangler config generated | Application compiles. Provider behavior and production routing remain unverified. |
| `npx tsc --noEmit --pretty false` | Passed | Current snapshot type-checks. Old TypeScript failure findings are no longer current. |
| `npm run lint` | **Failed: 11 errors, 35 warnings** | Current quality gate is red. See T01. |
| SQL migration replay | All 41 migrations apply to an empty SQLite database; integrity check `ok`, no foreign-key errors | Fresh SQLite schema is consistent. Existing production upgrade safety is a separate check. |
| `npm run dev:setup` | Passed against local D1 | Local migration and owner-seed flow completes. Generated credentials were kept out of the report. |
| Automated application tests | No test script, test suite, or `.github` workflow files found in the current tree | No current automated regression gate was available to execute. Old tests do not count as current evidence. |
| GitHub status | No commit statuses returned; main reported unprotected | External CI and repository rules still need a direct configuration check. This is not a full ruleset audit. |
| Production deployment, secrets, provider accounts | Not verified | T00 must establish deployed SHA, isolated staging, and actual integration readiness without exposing secret values. |

### Current inventory: reuse, finish, or build

Source paths below refer to the audited commit, not the older local checkout.

| Area | Current source evidence | Current assessment | Next work |
| --- | --- | --- | --- |
| Authentication and accounts | `app/native-auth.ts`, `app/access.ts`, `proxy.ts`, `app/api/auth/*`, `app/api/signup/route.ts`, `app/api/onboarding/route.ts`, `app/api/team/route.ts` | Native sessions, OAuth paths, invitations and onboarding exist. Access is still limited to a hard-coded approved-email set. | Verify trust boundaries; finish founder access and invitations. |
| CRM and sales | `app/api/crm/route.ts`, `public/sales-notes.js`, `public/pipeline-builder.js`, `public/views.js` | Persisted contacts, companies, pipelines, tasks, notes, call outcomes and audit records. New-contact handler now calls CRM APIs; old session-only contact finding is superseded. | Tenant/role tests, durable import, consistent navigation and timeline. |
| Calling and SMS | `app/api/twilio/*`, `lib/twilio*.ts`, `public/softphone.js`, `public/communications.js` | Real provider operations are implemented in source: SMS, voice, browser calling, callbacks, recording-choice flow and playback. Consent/suppression checks and STOP handling exist. | Verify current paths, tenant-specific numbers, retries, status ordering, recording access and delivery. No live test was performed. |
| Demo booking | `app/api/demo-bookings/route.ts`, `app/api/demo-bookings/[token]/route.ts`, `app/book/[token]/page.tsx`, `lib/demo-booking.ts` | SMS invitation and public booking route exist. Slots use fixed weekday hours, 30-minute duration and a two-hour lead time. | Configurable availability, cancellation/rescheduling, full reminders, general landing-page booking path. |
| Website forms | `app/api/website-connections/route.ts`, `app/api/v1/forms/[siteId]/route.ts`, `docs/website-form-api.md` | Authenticated website intake, routing, contact creation, deduplication and follow-up logic exist. | Verify and connect to a published form, reliable source attribution and booking. |
| Email correspondence | `app/api/v1/forms/[siteId]/correspondence/route.ts`, `integrations/google-workspace/Code.gs` | Correspondence ingestion and a Google Workspace relay exist. | Treat as a starting integration, not proof of a full synced inbox or outbound campaign service. |
| Google integrations | `app/api/google-connection/*`, `app/api/google-products/*`, `app/api/google-ads/*` | OAuth and refresh code exist. Gmail checks profile access; Products probes Calendar and Analytics access; Ads lists/selects accounts. | Actual synchronization/reporting and accurate capability labels. Ads request header defect described below. |
| Stripe, DNS, domains | `app/api/provider-connections/route.ts`, `public/provider-connections.js` | Encrypted connection handling and provider-specific reads/actions exist. Stripe product/payment reads do not populate verified subscription analytics. | Reconcile each UI claim with an actual operation; add billing events before revenue analytics. |
| Dashboard | `public/admin-dashboard.js`, `app/api/admin-insights/route.ts`, `app/api/sales-dashboard/route.ts` | CRM, demo, correspondence, manual purchase and manual feedback metrics. Several external metrics explicitly unavailable. | Replace stage proxies with definitions, product events, billing history and cohorts. |
| Marketing | `public/marketing.js`, `app/api/marketing/route.ts`, `app/api/marketing-scan/route.ts` | Saved idea boards, campaign/blog drafts, text-graphic PNGs and HTML markup scans. | Idea lineage, model generation, providers, publishing and outcome measurement. |
| Funnels and automation | `public/pipeline-builder.js`, `pipeline_automation_rules` in `db/schema.ts` | Draft rules and templates exist; rules remain paused. | Durable event execution, eligibility, retry/replay and genuine run history. |
| Support and surveys | `app/api/service/route.ts`, `service_tickets`, `customer_feedback` | Manual persisted tickets and check-ins. Support records use names/emails rather than a complete product-account relationship. | Inbox/widget intake, linked tasks, surveys and stage-specific experience. |
| SaaS Launchup operator billing | `app/api/operator/route.ts`, `platform_subscriptions`, `platform_subscription_events` | Platform-level manual subscription tracking is separate from tenant purchase records. | Keep this distinction; verify platform billing separately from a customer's SaaS billing. |
| Product telemetry, heatmaps and LLM | No complete collector, product-event pipeline, heatmap engine, grounded recommendation service or evaluation suite found | Major missing foundation | T22–T33. |
| Website builder, social/ad publishing, white label | Navigation/previews and partial connection work | Complete end-to-end capabilities not evidenced | Later tasks extend the working foundation. |

### Findings that change the execution order

**F01 — Current regression protection is missing.** Build and type checking pass, but lint fails and current main has no application test suite. Establish small critical-path tests before expanding side effects. Do not restore obsolete tests without adapting them to current routes/schema.

**F02 — Public founder access is still blocked by the access model.** `isApprovedLoginEmail` is consulted before workspace access, signup and recovery. Invitations for arbitrary customers therefore cannot be assumed usable. Reuse native authentication; replace the hard-coded business policy deliberately after isolation tests.

**F03 — Password setup has an identity trust boundary to resolve.** `proxy.ts` exempts `/api/auth/*`; `app/api/auth/setup/route.ts` calls `getChatGPTUser`, which reads identity headers in `app/chatgpt-auth.ts`. Establish whether a trusted gateway strips and signs/injects those headers for this route. Prefer a verified session or a short-lived verified setup grant. This is a source-level risk requiring a negative test, not a claim of a demonstrated production exploit.

**F04 — Journey semantics conflict with your latest vision.** Demo bookings and inbound sales email currently set Consideration. `advancePhase` only moves a contact forward, and `emailPhase` classifies transactional email as Purchase and cancellation email as Loyalty. An email is not payment evidence, a demo is not necessarily a trial, and an unhappy cancellation does not establish loyalty. Keep the pipeline state, subscription state, journey milestones and experience signals separate.

**F05 — Revenue language is stronger than the evidence.** The dashboard says “Verified purchases” for manually entered `purchase_records`. There is no contact/account foreign key in those records. Stripe reads and platform subscription records are separate. Label manual records clearly and require provider-backed billing evidence for verified revenue and trial-to-paid analytics.

**F06 — The requested reminder chain is not implemented in current main.** Public booking exists, but current code has no complete confirmation/reminder queue, dispatcher or reschedule/cancel lifecycle. The older dry-run appointment jobs are absent. Current uniqueness protects equal start times, not arbitrary overlapping intervals; cancelled appointments would also retain their unique start slot unless lifecycle/index behavior is redesigned.

**F07 — Some provider operations assume one deployment/account.** Demo invitation links hard-code `https://www.saaslaunchup.com`. A Twilio setup path references a fixed number and phone SID even though other number-selection paths exist. Replace these assumptions with validated environment/tenant configuration before onboarding other businesses.

**F08 — Google Ads account verification is incomplete.** `app/api/google-ads/route.ts` calls Google Ads with Authorization and Accept headers but no `developer-token`. The current official authorization documentation requires a developer token for Ads API calls [R1]. Implement server-side configuration and a tested provider adapter; OAuth success alone is insufficient.

**F09 — Provider readiness is not the same as a working feature.** Checking a Gmail profile, one Calendar event, Analytics account access, or listing Stripe payments is different from syncing an inbox, preventing booking conflicts, computing traffic metrics, or tracking subscriptions. Present credential, capability, sync and delivery health separately.

**F10 — Database changes need operational discipline.** Migrations `0026` and `0030` contain explicitly targeted historical data cleanup. They must not become examples for routine migration authoring. Audit applied migration history and backup/restore procedures; preserve already-applied migrations and use forward changes. Do not merge the old booking branch's conflicting migration numbering.

**F11 — Frontend behavior is spread across global scripts and embedded copies.** `templates/workspace.html` includes inline phone behavior alongside files such as `public/softphone.js`; `scripts/sync-phone-inline.mjs` exists to synchronize copies. Improve one module at a time and verify the script actually loaded by the workspace. Avoid a wholesale framework rewrite during workflow repairs.

## 3. Product definition Codex must implement

**Product promise:** SaaS Launchup helps a founder see where their customer journey loses value, understand the evidence, make an improvement through the CRM's tools, and measure whether that improvement produces more successful, retained paying customers.

**First customer:** a solo SaaS founder. **Later:** an agency managing isolated client workspaces with white-label presentation. A tenant is the founder's business/workspace; its SaaS product has customer accounts and end users. These end users are not SaaS Launchup staff memberships.

### The five stages and their boundaries

| Stage | Meaning for this build | Essential evidence and metrics | Features that act on the evidence |
| --- | --- | --- | --- |
| Awareness | Discovery and pre-trial acquisition: pages, ads, search, social, forms, prospecting and pre-trial demos | Sessions and views; engaged time; scroll reach; click concentration; form starts/submits/drop-off; source/campaign; qualified leads; call outcomes | Landing pages, forms, media, search/content, ads, sales scripts, booking |
| Consideration | Trial/free evaluation, onboarding and initial use inside the customer's SaaS | Trial starts/ends; activation; time to first value; feature use; onboarding abandonment; errors; help requests; trial activity by cohort | Onboarding flows, support, education, product improvement tasks and approved follow-up |
| Purchase | Decision and payment, including checkout friction and trial conversion | Checkout starts/failures/completions; paid conversion; time to payment; subscription changes; cancellation request versus effective cancellation | Checkout fixes, lifecycle messages, billing integrations and cancellation improvements |
| Experience | A cross-cutting view of what the customer experiences at every stage | Support reasons; response/resolution time; surveys; CSAT; user-reported obstacles; errors and outcomes | Unified support, surveys, bug/improvement tasks and educational content |
| Loyalty and advocacy | Sustained customer value, retention, expansion and voluntary recommendations | Repeat value events; customer/account retention; MRR movements; GRR/NRR; churn; education use; referral outcomes | Education, appreciation, success check-ins, referral programs and retention experiments |

Experience remains a visible dashboard tab, but its events also carry `stage_at_event`. Customers can skip a trial, return after churn, belong to several product accounts, or experience support before buying. Do not model the whole business as one irreversible five-value contact enum.

**Default interpretation:** a demo before a trial is an Awareness milestone; Consideration begins at `trial_started` or a tenant-configured evaluation event. Support may happen anywhere. A paid conversion requires the configured billing rule, not a pipeline move or the contents of an email.

### What every dashboard and AI recommendation needs

- Tenant, product, date range, timezone, relevant segment and cohort.
- Metric definition/version, source, last update, sample size and completeness/coverage.
- An explanation of the numerator, denominator and observation window; “unavailable” when there is no evidence, not a fabricated zero.
- A drill-down from summary to source events, contacts/accounts, forms, campaigns or tickets.
- Clear separation between observed behavior, a customer-reported reason, a model's hypothesis and a measured experiment result.
- An action that opens the relevant tool with a reviewable draft, then links the final action and its outcome back to the original finding.

### Minimum measurement rules

| Metric | Required definition |
| --- | --- |
| Activation rate | Unique eligible trial accounts satisfying a tenant-defined value event within an explicit window / eligible trial accounts in that cohort. Keep account and seat metrics separate. |
| Trial-to-paid conversion | Unique trial accounts with the configured first verified qualifying payment by the observation cutoff / eligible trial accounts. Show incomplete trial cohorts separately. |
| Checkout completion | Unique checkout sessions reaching verified success / eligible started sessions, with abandoned/failed/pending states and a documented timeout. |
| Engagement and bounce | Define active-time measurement and an engaged-session rule. Store the rule version. Do not compare custom bounce rates with GA4 rates as though definitions are identical. |
| Scroll and click heatmaps | Group by URL/page version, viewport class and element/section identifier. Scroll reach measures how many sessions reached a section; a stop point alone does not reveal why someone left. |
| Call script friction | Hang-ups or objections associated with a recorded script section / calls that actually reached that section. Track script version, audience, rep and sample size. Association is not causation. |
| MRR | Monthly-normalized recurring subscription value under a documented policy. Keep invoices/cash collected, one-off purchases, taxes, discounts, usage charges and currencies explicit. Never sum incompatible currencies. |
| MRR movements | Opening MRR + new + expansion + reactivation − contraction − churn = closing MRR, with adjustment explanations. |
| GRR / NRR | Starting-cohort recurring revenue retained excluding expansion / starting MRR; NRR includes that cohort's expansion. New customers do not enter the numerator. Zero denominator means unavailable. |
| Churn | Effective paid subscription/account churn under a tenant-selected rule. Cancellation requested, cancellation scheduled, account deleted and product inactivity are separate events. |
| Attribution | Record first touch, last eligible touch and the chosen conversion window. Do not invent cross-device identity or infer an individual organic search query from aggregate Search Console data. |

## 4. Architecture and sources of truth

Keep the current Next/Vinext, Workers and D1 stack for the first working release. Refactor only where a task needs it. Select additional event storage using measured event volume and query cost; do not put unbounded raw replay/media/transcript data into D1 by default.

| Data / capability | Source of truth and recommended direction |
| --- | --- |
| Workspace identity and authorization | Existing native sessions plus verified OAuth identities and tenant memberships; unify trusted identity handling. |
| CRM contacts, pipeline, notes and tasks | Tenant-scoped CRM database. Track product-account identities separately and link them explicitly. |
| Product behavior | Versioned browser/server SDK events from the customer's SaaS, with server-confirmed events for authoritative milestones. |
| Tenant SaaS billing | A tenant's connected billing provider; Stripe first because connection work already exists. Persist verified webhook events and reconcile with provider reads [R2]. |
| SaaS Launchup's own billing | Separate platform billing integration and entitlement model. Do not mix its MRR with customer SaaS MRR. |
| Voice/SMS | Existing Twilio implementation and authenticated callbacks, normalized into a shared delivery ledger. |
| Email | One transactional sender chosen/configured in T16; existing Gmail connection for mailbox work is a separate capability. Campaign eligibility and deliverability need separate controls. |
| Scheduling | SaaS Launchup appointment state with external-calendar mappings; explicit authority for changes and conflict reconciliation. |
| Support | Unified ticket/conversation records linked to product accounts, contacts, events and tasks. |
| Organic search | Authorized Search Console property data, plus page audits and separately labeled mention observations [R3]. |
| Ads | Provider-reported spend/delivery, reconciled with first-party conversion evidence; retain provider attribution separately. |
| AI analysis | A server-side model gateway using deterministic metrics and retrieved business/playbook evidence. The model is not the source of financial or usage truth. |
| Assets and publishing | Versioned drafts/assets, explicit destination accounts, reviewed publish jobs and provider results. |

### Shared contracts before adding more dashboards

The names below are proposed contracts; reuse existing tables where their semantics match.

- `product`: belongs to a tenant; owns domains, journey definitions, event schemas and integrations.
- `product_account`, `product_user`, `identity_link`: map external IDs to CRM entities. Do not silently merge people solely because a browser changes email; preserve provenance and reversal.
- `journey_definition`: activation event/predicate, trial policy, paid rule, inactivity threshold, churn rule, attribution window, timezone, currency and effective version. Each tenant can configure these.
- `journey_event`: `event_id`, schema version, tenant/product derived from authentication, external user/account/anonymous IDs, event name, occurred/received timestamps, session, feature, page/campaign/variant IDs and allowlisted properties. Deduplicate by tenant + source + event ID.
- `delivery_job` and `delivery_attempt`: purpose, channel, destination reference, policy result, schedule, idempotency key, provider ID, attempts and final outcome. Record uncertain outcomes for reconciliation rather than blindly retrying a possibly successful send.
- `metric_snapshot`: query/definition version, cohort, interval, timezone, source watermark and result. The LLM receives approved metric output instead of writing unrestricted SQL.
- `recommendation`, `action_draft`, `experiment`, `outcome`: preserve evidence, hypotheses, approvals, exact content versions, exposure assignment and measured results.

Browser collection uses a public project identifier with ingestion-only permissions, quotas, origin checks and abuse controls. A public key is not a secret and cannot verify payment or trial status. Private API secrets and billing credentials never go in browser JavaScript. Minimize captured properties, mask sensitive inputs by default, and support tenant retention, export and deletion rules as functional requirements.

### AI design: build expertise you can inspect

Start with a capable model plus curated, versioned customer-journey playbooks, tenant business context and permitted tools. Fine-tuning is a later option after there is a reviewed example set and an evaluation result showing a specific gap. Do not start by claiming to train a model to know everything.

Each recommendation must contain:

1. The business question, stage, segment and date/cohort window.
2. Evidence IDs and metric snapshots, with source freshness and sample size.
3. What is observed, the proposed explanation, uncertainty and alternative explanations.
4. A suggested action inside Marketing, Sales, Support, Sites or Automations.
5. Expected outcome as a hypothesis, effort, primary metric and guardrails.
6. Draft/action/experiment IDs and the follow-up result, including no improvement.

Prioritization should explain impact, confidence, effort and missing evidence. Low-data mode should propose interviews, reviews or instrumentation instead of inventing statistical certainty. Page copy, email, transcripts and retrieved documents are untrusted evidence; they cannot grant tool permissions or override system rules. Tenants' data and knowledge retrieval must remain isolated.

The innovative loop is: **find a bottleneck → explain the evidence → create a reviewed change → track exposure → measure outcome → update the playbook**. The home dashboard eventually answers “What should I improve next?” with links to the five stage views. It should connect the promise made in an ad with the value actually delivered in the product.

## 5. Release gates and execution order

The default order is T00 through T66 below. Dependencies are explicit where another lane could be worked later; one executing Codex task still owns one task ID. A blocker affects dependent work, not unrelated safe documentation or local work.

| Gate | Tasks | Completion evidence | Release meaning |
| --- | --- | --- | --- |
| G0 — Reconciled and testable | T00–T07 | Deployed/source mapping, green basic checks, isolated staging, trusted identity, tenant boundaries, accurate capability labels | Safe foundation for continuing development |
| G1 — Usable acquisition workflow | T08–T21 | Real staging lead/call → booking → confirmations/reminders; opt-out and reschedule tests; support → task; surveys | Assisted solo-founder beta is possible; only completed capabilities are advertised |
| G2 — Trustworthy journey data | T22–T29 | One customer SaaS instrumented; known trial/activation/payment/cancellation fixtures reconcile to dashboards | The five-stage analytics promise becomes usable |
| G3 — Evidence-backed AI improvement | T30–T34 | Grounded recommendations, reviewed drafts, experiment results and reliable automation runs | Differentiated growth-assistance beta |
| G4 — Distribution and creation | T35–T59 | Each provider/content/builder capability proven separately | Broader marketing, site building, ad and calendar offering |
| G5 — Retention and agency scale | T60–T65 | Loyalty workflows, reusable templates, tested agency isolation and white-label provisioning | Phase-two agency offer |
| G6 — Broad launch | T66 | Operational, accessibility, security, cost and end-to-end release evidence | Wider self-service release of completed modules |

Do not wait for the advanced canvas, video generation, every ad network or white label to learn from the first founder. Do not advertise unfinished capabilities as included working tools.

## 6. Execution rules for every task

- Read the current repository instructions, this plan and current implementation evidence before editing. Record the actual starting SHA; this report's SHA is a baseline, not a permanent target.
- Reuse current routes/schema and preserve customer data. Proposed paths in later tasks can be adjusted with a short explanation.
- Keep changes within the selected task. If a task genuinely needs multiple independently reviewable slices, name them and implement only the first slice; do not hide unrelated feature work in it.
- Use local/staging providers and synthetic records for verification. Never run development seeds or cleanup against production.
- Implement server authorization and tenant filters, input validation, loading/empty/error states, and audit/provenance where the feature needs them.
- Test meaningful failure cases: unauthorized tenant/role, retries, duplicates, stale versions, partial provider failure and insufficient evidence. Do not add tests that merely mirror presentation copy.
- After a mutation, verify persisted/backend state as well as the visible UI. A button, saved credential, accepted send or screenshot alone is insufficient proof of a complete workflow.
- Provider changes, publishing and paid operations require the application's appropriate approval/policy gates. Development agents must not send to real prospects or spend money just to demonstrate a task.
- Record actual commands/results, runtime evidence and remaining blockers in `docs/implementation-status.md` and `docs/evidence/Txx.md`. Keep secrets and customer data out of these files.
- End each task with changed files, behavior, verification, limitations, rollback/migration notes and the next eligible task. Do not automatically execute the next ID.

All acceptance criteria below supplement these common rules.

## 7. Task-by-task handoffs

### T00 — Reconcile the live deployment and establish the working baseline

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** None.
- **Inspect first:** Current main, README.md, .openai/hosting.json, vite.config.ts, scripts/setup-local.mjs; compare this audit with the actual deployed build.
- **Execute:** Record current SHA, deployment owner/platform, production and staging URLs, database bindings, applied migration ledger and provider capability inventory without secret values. Create/update docs/implementation-status.md and docs/evidence/T00.md. Inspect each authenticated menu in staging with an approved test account; record implemented, preview, broken or blocked. Identify release SHA through deployment metadata, not visual similarity. Record the old branch divergence and canonical booking implementation.
- **Done when:** A reproducible local start and isolated staging are documented. Every menu has evidence and an owner/task. If production login or hosting access is unavailable, record the exact missing access; do all accessible work and mark only that sub-check blocked. No old audit result is presented as current.

### T01 — Make lint a usable quality gate

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T00.
- **Inspect first:** package.json and the lint output; app/foundation/page.tsx, app/reset-password/page.tsx and page components with internal anchor links.
- **Execute:** Fix the 11 current lint errors, preserving routing, cookie checks and password-reset expiry behavior. Resolve warnings in touched code and list the remaining warning baseline. Address the reset-page clock/purity issue deliberately; do not weaken expiry checks. Do not disable lint rules globally to obtain a pass.
- **Done when:** npm run lint exits 0; type check and build pass. Login, recovery and booking navigation still work in local/staging. Old type errors are not reintroduced. No speculative UI rewrite.

### T02 — Restore critical-path tests and CI

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T01.
- **Inspect first:** package.json; current auth/CRM/form/demo-booking/Twilio routes; current main has no test suite.
- **Execute:** Add documented unit, D1 route and browser smoke test commands, plus CI for lint, type check, build and critical tests. Begin with auth denial, tenant denial, contact save/reload, duplicate form ingestion and single-slot booking. Mock outbound providers by default. If recovering old tests, port them to demo-bookings and the current schema. Add repository required-check instructions; change repository settings only within deployment authorization.
- **Done when:** A clean CI environment installs its needed browser/runtime and executes tests. Tests prove failure before the relevant defect is fixed where applicable. No test can accidentally call or message an external customer. Publish a small current test evidence report.

### T03 — Make schema changes and recovery reproducible

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T02.
- **Inspect first:** drizzle/*.sql, drizzle/meta/_journal.json, scripts/setup-local.mjs and actual staging migration history.
- **Execute:** Check fresh install and an upgrade from the currently deployed schema. Inventory targeted cleanup migrations without replaying them against production. Document schema ownership, backup/restore, forward-migration practice and rollback limits. Keep the old 0016 booking branch out of the current chain. Add a staging restore rehearsal and a migration gate.
- **Done when:** Fresh and upgrade paths preserve representative contacts, memberships, consent, calls and bookings. Restore is exercised on isolated data. The agent can show which migration will run before a deploy. Production IDs or deletion instructions are not copied into new generic migrations.

### T04 — Resolve authentication trust boundaries

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T02,T03.
- **Inspect first:** proxy.ts, app/chatgpt-auth.ts, app/native-auth.ts, app/api/auth/setup/route.ts and other proxy-exempt routes.
- **Execute:** Make password setup require a verified identity/session or signed short-lived setup grant. Audit every bypassed route for its own auth, signature or scoped-key check. Prove externally supplied identity headers cannot authorize requests. Verify session revocation, reset token single use, origin checks, expiry, OAuth state/PKCE and rate limits. Keep the existing private access boundary until replacement behavior is tested.
- **Done when:** Unsigned/spoofed identity, replayed reset/setup grants, expired sessions, wrong origins and mismatched OAuth states fail in local/staging. Legitimate login/setup/recovery succeeds. Record any remaining trusted-gateway assumption and how it is enforced.

### T05 — Finish solo-founder onboarding and usable invitations

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T04.
- **Inspect first:** app/access.ts, app/api/signup/route.ts, app/api/onboarding/route.ts, app/api/team/route.ts and signup/onboarding pages.
- **Execute:** Replace hard-coded approved-email business logic with an explicit launch policy: approved beta invitations first, self-service later. Make verified new identity → workspace → owner membership atomic and retry-safe. Fix any onboarding-versus-membership gate deadlock. Reuse saved drafts and invite flows. Start with business/product name, site and timezone; request extra company/phone-verification information only when the relevant feature needs it.
- **Done when:** A newly invited founder with no existing membership completes onboarding and re-enters the same workspace. Duplicate submission does not create duplicate businesses. Invitations, revocation, owner transfer and recovery work for non-hard-coded test identities. Onboarding measures its own completion separately from the customer's SaaS trial.

### T06 — Prove tenant, role and operator isolation

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T05.
- **Inspect first:** lib/workspace-permissions.ts, API authorization helpers, record scopes, app/api/operator/route.ts and frontend tenant selection.
- **Execute:** Create two synthetic tenants and representative owner/admin/sales/marketing/support users. Centralize active tenant selection and use it consistently across modules. Test contacts, notes, tasks, recordings, providers, bookings, private API tokens and operator inspection. Eliminate inconsistent selection of a user's first organization. Require reason/audit for privileged operator inspection.
- **Done when:** Cross-tenant IDs and lower-privilege roles cannot read or mutate forbidden records. Switching tenants clears cached UI data. A stale response cannot populate another tenant's screen. Tenant owner and platform operator permissions remain distinct.

### T07 — Align UI capabilities and remove misleading claims

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06.
- **Inspect first:** public/app.js, public/views.js, public/admin-dashboard.js, templates/workspace.html, public/softphone.js and scripts/sync-phone-inline.mjs.
- **Execute:** Make each menu's actual state visible: available, setup required, draft only or planned. Rename manual purchases/subscriptions as manually recorded. Correct stale README claims about authentication/calling. Map every screen to its actual handler and remove duplicate event execution where embedded/source copies conflict. Check the Google sign-in contrast, keyboard focus, form errors and small-screen navigation.
- **Done when:** All advertised actions either persist/perform their named operation or clearly explain their current limit. Contact save survives reload. One click produces one handler/action. Manual entries are never displayed as provider verified. Accessibility checks cover entry and primary workflow pages.

### T08 — Define configurable journey rules and reconcile old classifications

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T07.
- **Inspect first:** lib/customer-journey.ts, contacts.customer_phase, correspondence and demo-booking phase writes, public/admin-dashboard.js.
- **Execute:** Create versioned tenant/product journey definitions for activation, trial, payment, churn, inactivity and attribution. Separate pipeline stage, product lifecycle, subscription state and stage-at-event. Implement this document's stage defaults, including trials under Consideration and Experience at all stages. Prepare a previewable reclassification of old events; preserve original labels/provenance.
- **Done when:** Demo booking alone does not become a trial; transactional email does not verify purchase; cancellation mail does not imply loyalty. Direct-to-paid, reactivated and multi-account customers are representable. Existing records migrate without erasing history; definitions are editable and versioned.

### T09 — Unify communication eligibility and opt-outs

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06,T08.
- **Inspect first:** consent_records, suppression_records, Twilio send/booking routes, inbound STOP handling and communications UI.
- **Execute:** Preserve existing working checks while introducing one server-side eligibility service with channel, purpose, scope, contact/address, timestamp and evidence. Evaluate it at dispatch, not only when drafting. Cover unknown inbound numbers and future contact merges so opt-outs survive identity changes. Model permission evidence and contact validity separately. Keep transactional and marketing policies distinct.
- **Done when:** SMS STOP blocks subsequent queued messages; revoked permission or DNC blocks affected actions; email suppression is independent of phone permission. Missing evidence produces a clear reason. Importing a contact or scheduling a demo does not silently opt them into marketing.

### T10 — Make integration readiness truthful and tenant-specific

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06.
- **Inspect first:** app/api/integrations/route.ts, app/api/provider-connections/route.ts, app/api/google-* and provider UI.
- **Execute:** Create a capability inventory: credential stored, authorized scope, selected asset/account, operation ready, last successful sync/delivery and error. Add missing Google Ads developer-token configuration and applicable manager-account context using current official docs [R1]. Verify refresh/reconnect, disconnect, secret rotation and access scoping; never return secrets to the browser.
- **Done when:** Provider mocks prove required headers and refresh failures; a configured staging account can demonstrate the specific capability. A connected badge cannot imply email sending, calendar writes, analytics reports or ads publishing. No credentials/account access are fabricated when unavailable.

### T11 — Verify website and ad-lead intake end to end

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T08,T09,T10.
- **Inspect first:** app/api/v1/forms/[siteId]/route.ts, website-connections, Google Ads lead-form route and docs/website-form-api.md.
- **Execute:** Reuse the existing keyed webhook. Exercise field mapping, tenant routing, owner selection, duplicate matching, task creation and request limits. Preserve source/campaign/form/page version and consent evidence. Provide a safe server relay or publishable intake path for browser forms; never expose the existing private key in page source. Link the success state to the appropriate booking flow.
- **Done when:** A synthetic published form creates one persisted lead and next task, retries deduplicate, invalid auth/routing fails, and another tenant cannot inject records. Source metadata survives into the contact timeline. Form submission alone does not authorize unrelated outreach.

### T12 — Turn contact import into a real reviewed operation

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06,T09.
- **Inspect first:** public/views.js renderImport, CRM contact creation and lib/contact-channels.ts; current CSV flow is preview-only.
- **Execute:** Implement robust CSV parsing, field mapping, row validation, duplicate preview, owner assignment and create/update/skip decisions. Add a bounded import job with per-row results and idempotency; retain provenance. Do not treat an imported permission column as consent without its required evidence. Keep campaigns off during import.
- **Done when:** Quoted commas/newlines and malformed rows are handled; repeating an import does not duplicate approved rows; tenant/role limits hold. Counts match persisted records and failures are exportable. Importing never starts calls, email or SMS.

### T13 — Verify and harden current Twilio calling/SMS

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T09,T10.
- **Inspect first:** lib/twilio.ts, lib/twilio-browser.ts, app/api/twilio/*, phone settings, softphone and recording routes.
- **Execute:** Remove fixed-number/SID assumptions from production paths and select tenant-owned, authorized lines. Test browser/manual calls, incoming routing, recording acceptance/decline, playback access, SMS and provider callbacks with mocks and a designated staging test recipient when authorized. Add request idempotency and reconcile ambiguous provider responses. Prevent old status callbacks from regressing terminal delivery state.
- **Done when:** Two workspaces use different lines safely; repeated clicks/retries produce no unreviewed duplicate send; invalid webhook signatures fail. A recording never starts before the implemented acceptance flow and is visible only to permitted members. Staging evidence distinguishes queued, delivered, failed and unknown.

### T14 — Extend existing demo booking with real availability

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06,T10.
- **Inspect first:** lib/demo-booking.ts, demo_appointments, demo_booking_invites, app/book/[token]/page.tsx and both demo-booking routes.
- **Execute:** Reuse token-based booking. Replace fixed hours/duration/lead time with per-owner timezone schedules, buffers, exceptions and limits. Replace the production-hard-coded booking origin with validated environment configuration. Add a safe public booking entry after a form, without requiring an SMS send. Enforce overlap under concurrency at the database/service boundary.
- **Done when:** Timezone/DST, no available slot, expired token, adjacent slots and competing bookings are tested. One conflicting request succeeds and one fails cleanly. Public output reveals only necessary booking information. Existing invitation links remain usable under the migration policy.

### T15 — Add cancellation, rescheduling and appointment outcomes

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T14.
- **Inspect first:** Current demo-booking schema/routes and Calendars views; create lifecycle/version fields through forward migrations.
- **Execute:** Add authenticated and scoped public reschedule/cancel flows, optimistic version checks, no-show/completed outcomes and timeline/audit events. Make cancelled slots reusable while preventing overlapping booked intervals. Define external-event mapping fields now without pretending sync exists.
- **Done when:** Old links/versions cannot revive cancelled appointments. Repeated requests are safe. Concurrent cancel/reschedule resolves predictably. A moved appointment frees the old slot, and the UI preserves entered data through slow requests.

### T16 — Implement one transactional email adapter

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T09,T10.
- **Inspect first:** message_templates, existing email drafts and Google connection capabilities; select a sender with documented API and delivery callbacks.
- **Execute:** Choose/configure one transactional sender, recording why and its domain requirements. Implement server-side templated confirmations with provider IDs, delivery/bounce/complaint callbacks, suppression, retry/reconciliation and a staging sink. Reuse template data. Treat Gmail read authorization as a separate capability; do not call it a sending service.
- **Done when:** A message can be previewed, accepted and reconciled to delivery/failure; invalid callbacks fail. A sender outage preserves a recoverable job and does not claim delivery. Template variables are escaped, and customer data/secrets stay out of logs.

### T17 — Create the exact confirmation and reminder schedule

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T13,T15,T16.
- **Inspect first:** Current demo appointments; add a shared delivery-job/outbox model because old appointment_reminder_jobs are absent from current main.
- **Execute:** On booking, create immediate email and SMS confirmations; email+SMS 24 hours before; email+SMS four hours before; SMS 15 minutes before. Schedule in UTC using the appointment's timezone for display. Key jobs by appointment/version/purpose/channel. Invalidate old jobs on reschedule/cancel, skip elapsed reminders and re-check eligibility at delivery. Include calendar attachment/link where supported.
- **Done when:** A future appointment creates seven intended jobs. A last-minute booking omits past reminders. Cancellation creates no further appointment reminders; rescheduling creates only the new version. Job creation and appointment state are transactionally consistent.

### T18 — Dispatch reminders reliably and expose delivery history

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T17.
- **Inspect first:** Delivery jobs from T17 and current Workers runtime; use a durable queue or scheduled worker with atomic claims.
- **Execute:** Implement leases, bounded retries/backoff, dead-letter/review state, stale-version checks, provider reconciliation and a workspace kill switch. Record each attempt, consent outcome and provider receipt. Add calendar/contact run history. Avoid promising exactly-once delivery across the provider boundary; handle uncertain sends explicitly.
- **Done when:** Two workers do not claim the same active lease. A crash/retry, provider timeout, cancellation during dispatch and opt-out before send all have tested outcomes. The UI shows scheduled, skipped, attempted, accepted, delivered, failed or uncertain states truthfully.

### T19 — Complete and release the first acquisition workflow

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T11,T12,T13,T18.
- **Inspect first:** Sales call outcomes/scripts, forms, CRM timeline, booking pages and delivery history.
- **Execute:** Connect called → outcome → demo invitation/booking → confirmations → reminders → completed/no-show → follow-up task. Also prove landing page/form → contact → booking. Reuse each existing screen and remove dead-end transitions. Add error monitoring and a small assisted-beta release checklist.
- **Done when:** One synthetic contact can be traced across the whole workflow after reload. Verify all seven reminder timings with a controllable clock, duplicate intake, opt-out, reschedule/cancel, wrong tenant and provider failure. No real prospect is contacted for testing. Publish evidence before calling this workflow working.

### T20 — Close the support-to-improvement loop

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06,T08.
- **Inspect first:** app/api/service/route.ts, service_tickets, customer_feedback, CRM tasks and support UI.
- **Execute:** Link tickets to contact/product account, stage, feature, source event and optional subscription. Add conversation entries, assigned owner, reason categories and timestamps. Implement create/link improvement task with a backlink and resolution evidence. Keep cancellation request handling separate from actually cancelling billing.
- **Done when:** A support case creates a linked task without duplication; reopening and closing retain history. Trial support appears in Experience and its trial context. A resolved cancellation ticket cannot silently change provider billing. Permission and two-tenant tests cover the full loop.

### T21 — Capture surveys and feedback throughout the journey

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T20.
- **Inspect first:** Current manual feedback schema and forms; extend with response, survey version, trigger and account linkage.
- **Execute:** Build a small survey composer/template set for onboarding obstacles, trial exit, support satisfaction and cancellation reasons. Trigger only approved surveys and deduplicate invitations/responses. Record incentives separately; reward participation rather than positive sentiment. Add opt-out, expiry and aggregate response rates.
- **Done when:** Feedback is linked to its exact question/version, stage and triggering event. Nonresponse is not classified as satisfaction. Survey responses create reviewable themes/tasks without automatically asserting a reason for every abandonment. Incentivized responses remain identifiable in analysis.

### T22 — Define product identities and the event contract

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T08,T19.
- **Inspect first:** db/schema.ts, website intake, CRM activities and private-token scopes; proposed contracts in section 4.
- **Execute:** Add tenant product, external account/user and identity-link models plus a versioned event schema. Define dedup keys, occurred/received time, bot/internal/test exclusions, environments and schema validation. Classify client observations versus authoritative server/billing events. Establish property allowlists, retention, deletion/export and a volume/cost budget before selecting raw-event storage.
- **Done when:** A reviewed fixture covers anonymous visitor → known contact → trial account with two seats → paid account. Duplicate, late, out-of-order and invalid events have specified outcomes. Tenant and production/test streams cannot mingle. A customer can be linked without inventing cross-device identity.

### T23 — Implement authenticated event ingestion and data health

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T22.
- **Inspect first:** Reuse API-token patterns and form ingestion conventions; add product-event routes, storage and background processing.
- **Execute:** Accept bounded batches through scoped credentials, rate limits and schema checks. Derive tenant/product from credentials. Deduplicate and quarantine invalid events; preserve source timestamps and provenance. Add a test-event console, ingestion latency/error counters, retention processing and replay controls.
- **Done when:** Retries do not inflate counts; cross-tenant project IDs cannot redirect events; late events update the appropriate cohort. Forged browser payment events cannot change billing truth. Health reports distinguish no traffic, missing SDK, rejected schema and processing delay.

### T24 — Build the first-party website tracker and heatmaps

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T23.
- **Inspect first:** New versioned browser SDK; existing website connections and future page builder use the same collector.
- **Execute:** Collect page/session views, active engagement time, form-step events, scroll reach and selected element clicks. Support SPA navigation, page versions, viewport classes and internal-traffic exclusions. Mask inputs/text by default, avoid keystroke collection, respect the configured tracking policy, and include sampling/kill switches. Render aggregate heatmaps against the correct page/viewport version.
- **Done when:** Known synthetic visits reconcile with counts and heatmaps; hidden tabs do not inflate active time; page revisions do not mix coordinates. Consent-disabled sessions and sensitive form fields are excluded as configured. Collector failure does not break the customer's website. Bounce and engagement definitions are visible.

### T25 — Instrument one customer's trial and product-value path

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T23.
- **Inspect first:** New server/client SDK examples and one designated test SaaS integration.
- **Execute:** Ship documented identify, account/group, track and error APIs. Map trial_started, onboarding_step, activation/value events, feature use, help request, inactivity and deletion using the tenant definition. Capture sanitized feature/error context and session duration; correlate with existing support. Use server events for trial status and trusted lifecycle transitions.
- **Done when:** One real staging SaaS produces a traceable trial journey without manual database edits. Feature usage and errors appear for the right account/seats. No tokens, payment fields or arbitrary sensitive payloads are collected. Exiting a feature is recorded as behavior, not automatically labeled dislike or a bug.

### T26 — Implement deterministic funnel and cohort metrics

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T24,T25.
- **Inspect first:** Event store, journey definitions and dashboard APIs; add a tested analytics service.
- **Execute:** Compute Awareness and Consideration metrics with explicit denominators, windows, active-time rules, cohort maturity and definition versions. Build source → form → trial → activation views, feature adoption and time to first value. Support segment filters, data quality and drill-down. Preserve unmapped/unknown sources rather than allocating them artificially.
- **Done when:** Seeded fixtures produce exact expected counts/rates, including repeat visitors, several seats, direct trials, bot exclusions and late events. Immature cohorts are flagged, not compared as final results. The same query returns the same result outside the LLM.

### T27 — Connect tenant SaaS billing as verified evidence

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T10,T22.
- **Inspect first:** Existing Stripe connection/payment reads, purchase_records and current absence of a subscription event pipeline.
- **Execute:** Add a tenant-scoped Stripe billing adapter, signed webhook validation, event-ID deduplication, reconciliation/backfill and external account/customer/subscription mappings. Handle trial, payment success/failure, refunds, scheduled/effective cancellation, plan changes and reactivation. Define qualifying paid conversion and historical-import policy. Preserve manual records separately. Use official subscription/webhook guidance [R2].
- **Done when:** Sandbox fixtures reconcile to the provider across duplicates and out-of-order delivery. Pipeline wins and browser events cannot mark a verified payment. Tenant SaaS subscriptions never update platform_subscriptions. The cancellation state matches provider effective timing and payment failures are distinguishable.

### T28 — Calculate MRR, retention and revenue movements

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T26,T27.
- **Inspect first:** Billing event history, product accounts and metric definitions in section 3.
- **Execute:** Build monthly-normalized MRR and movement ledgers, account/user retention cohorts, GRR/NRR, activation-to-retention and repeat-value metrics. Make currency, annual plans, discounts, usage billing and trial/free exclusions explicit. Preserve point-in-time history; show cash/revenue amounts separately from MRR.
- **Done when:** Opening + movements reconciles to closing MRR on fixtures with annual plans, partial cancellations, upgrades and reactivation. GRR/NRR use the starting cohort and handle zero denominators. A silent user is not automatically churned; a cancelled-at-period-end subscription remains classified by the chosen effective policy.

### T29 — Replace all five dashboards with connected evidence

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T20,T21,T26,T28.
- **Inspect first:** public/admin-dashboard.js, app/api/admin-insights/route.ts and app/api/sales-dashboard/route.ts.
- **Execute:** Build each stage view using the metric service, retaining useful CRM reports. Add cohort/date/timezone/segment controls, source freshness, confidence/sample size, unavailable states and drill-downs. Put trial/product usage in Consideration and experience evidence across stages. Keep platform owner reports distinct from customer SaaS analytics.
- **Done when:** A fixture story can be followed from source to trial, payment, support and retention with matching totals. Changing a filter affects all relevant cards consistently. Manual/provider metrics are labeled and not silently summed. Dashboard queries meet a documented latency/cost budget.

### T30 — Create the AI gateway, business context and knowledge layer

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T29.
- **Inspect first:** No complete LLM integration found; build a server-side gateway with an explicit model/provider configuration.
- **Execute:** Store tenant product value proposition, ideal customers, pricing/trial rules, goals, brand constraints and approved playbooks. Add versioned prompts, retrieval filtered by tenant/product, deterministic metric tools and usage limits. Log redacted tool/evidence references and model/prompt versions. Treat website/email/transcript text as untrusted. Start with retrieval and evaluations; defer fine-tuning unless measured need emerges.
- **Done when:** The model can answer a test business question using permitted facts and cite internal evidence IDs. It cannot retrieve another tenant's context, expose credentials or execute arbitrary SQL/tool actions. Missing integrations/data produce an explicit limitation. Cost and latency per request are visible.

### T31 — Deliver evidence-backed growth recommendations

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T30.
- **Inspect first:** Recommendation contract in section 4, metrics and tenant playbooks.
- **Execute:** Implement stage-specific and overview recommendations with observed facts, hypotheses, alternative explanations, priority rationale, action destination and measurement plan. Add low-data mode and a reviewed evaluation set covering fabricated numbers, causality errors, incomplete cohorts, conflicting sources, prompt injection and cross-tenant retrieval. Display model limitations in context rather than generic disclaimers.
- **Done when:** Each recommendation's quantitative statement resolves to a metric snapshot. Missing/contradictory evidence cannot produce a confident diagnosis. An evaluation rubric and threshold are recorded and passed before release. No automatic claim that a color, script phrase or feature caused the outcome.

### T32 — Turn recommendations into reviewed action drafts

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T31.
- **Inspect first:** Existing marketing documents, scripts, CRM tasks, support tasks and automation drafts.
- **Execute:** Add Create draft actions for copy, form questions, call-script variants, lifecycle messages, support improvements and experiments. Carry product/segment/evidence/version context into the destination so the founder does not re-enter it. Show an exact diff/preview and intended audience/destination. Require the appropriate application review for sends, publishes and paid changes.
- **Done when:** An insight creates an editable draft linked back to its evidence, without automatically sending or spending. Approval is tied to the exact version; editing material content invalidates old approval. A missing permission or provider blocks execution with a useful next step.

### T33 — Measure experiments and learn from results

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T32.
- **Inspect first:** Recommendation/action links, first-party events and proposed experiment/outcome models.
- **Execute:** Implement stable assignment at the chosen account/user/session unit, exposure logging, primary metric, guardrails, duration/stopping plan and immutable variant versions. Support landing copy/form questions/onboarding first. Account for delayed revenue and retention outcomes. Keep inconclusive and negative results. Add a marketing-promise versus delivered-value report as a hypothesis generator.
- **Done when:** A seeded experiment reproduces allocation and exposure counts, detects sample-ratio/data-quality problems and never announces a winner from insufficient evidence. Results link to the exact content version and metric definition. Nonrandom before/after comparisons are labeled observational.

### T34 — Run general automations using the shared job system

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T18,T23,T32.
- **Inspect first:** pipeline_automation_rules, public/pipeline-builder.js and the delivery infrastructure from reminders.
- **Execute:** Implement versioned trigger → conditions → action execution for approved form, pipeline, trial and support workflows. Use durable events/outbox, deduplication, loop limits, retry/review states, scheduling, suppression and execution history. Reuse reminder delivery infrastructure rather than introducing a competing scheduler. Provide simulation, pause and safe replay.
- **Done when:** A duplicate trigger creates one intended run; a loop is stopped; a delayed action sees current permission/appointment state. Every step shows input/output/error provenance without secrets. Draft rules remain inactive until validated and enabled. Existing reminders continue to work.

### T35 — Launch reviewed email campaigns from ideas

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T16,T32,T34.
- **Inspect first:** Email marketing/templates/consent UI, marketing documents and shared delivery system.
- **Execute:** Create idea → campaign brief → segment → template variants → review → scheduled campaign. Add subscription preferences, unsubscribe, bounce/complaint feedback, rate limits and domain readiness. Define a separate outreach policy for cold campaigns and gate eligibility accordingly; do not assume all imported addresses qualify or every sender permits the use case. Track replies and downstream trials/payments, not opens alone.
- **Done when:** Preview counts, exclusions and consent basis match the send ledger. A campaign can be paused and resumed without duplicate sends. Test-only recipients demonstrate delivery and opt-out. Every sent variant links to its idea, audience and outcomes.

### T36 — Build the connected email/support inbox

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T20,T10,T16.
- **Inspect first:** Existing Gmail OAuth/profile checks, correspondence API and Google Workspace Apps Script relay.
- **Execute:** Choose one first mailbox provider and implement incremental sync/backfill, thread/message deduplication, contact/account matching, attachments handling, reply permissions and support assignment. Avoid importing the same message through both relay and direct sync without a common identity. Treat sender authorization and mailbox read permission separately. Link support conversations to tickets.
- **Done when:** Existing and new messages reconcile into threads; a reconnect resumes safely; duplicates and moved/deleted items have documented behavior. A reply goes only to the reviewed thread/recipients. Restricted users cannot read another tenant's inbox or attachment.

### T37 — Add Search Console reporting to Awareness

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T10,T26.
- **Inspect first:** Existing markup scan and Google OAuth patterns; add a separate authorized Search Console property capability.
- **Execute:** Connect verified properties, sync query/page/device/date metrics, retain refresh watermarks and expose clicks, impressions, CTR and average position where returned. Keep the markup scan as a separate technical assessment. Handle provider limits and incomplete data; use the current Search Analytics API [R3]. Link opportunities to editable content ideas.
- **Done when:** Fixture and authorized test-property results reconcile to returned API rows. No universal rank or GEO/AEO score is invented from page markup. Search queries remain aggregate unless actual first-party attribution exists. A stale connection is clearly marked.

### T38 — Build the in-house mention monitor

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T10,T26.
- **Inspect first:** No complete mention collector found; design the service independently from the CRM forms API.
- **Execute:** Add tenant brand/domain/competitor query configuration, approved source adapters, scheduled collection, source URL/timestamp, deduplication, relevance review and coverage reporting. Start with one permitted search/source API, then add Reddit or other sources only with documented access. Store observations and quoted snippets within source limits. Distinguish web mentions, search ranking and observed AI-answer citations.
- **Done when:** Duplicate pages do not inflate mentions, ambiguous brand matches can be rejected, unavailable sources show gaps and source links support findings. The product never promises to scrape the whole internet or equates no collected mentions with no mentions existing.

### T39 — Implement Google Ads reporting and conversion joins

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T10,T26,T27.
- **Inspect first:** Existing Google Ads account selection and lead-form intake; Ads navigation.
- **Execute:** Add campaign/ad/ad-group reporting, pagination, spend/currency/timezone normalization, incremental sync and selected-account validation. Retain IDs for creative/landing/form versions and eligible click identifiers. Join first-party lead/trial/payment outcomes where supported, showing provider-attributed and first-party-attributed numbers separately.
- **Done when:** Reports reconcile with controlled provider fixtures and an authorized test account. Retries do not double spend totals. Unmatched conversions stay unattributed. No campaign creation or budget change is part of this read-only reporting task.

### T40 — Add Meta Ads reporting

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T39.
- **Inspect first:** Reuse the provider capability, sync and reporting contracts; a Facebook sign-in identity is not an Ads account integration.
- **Execute:** Implement the current approved Meta Marketing API authorization, business/ad-account selection, token lifecycle and read-only campaign/creative/insight sync. Add permissions/app-review readiness and provider error states. Map common metrics without forcing incompatible attribution windows into the Google schema.
- **Done when:** A test account or recorded fixtures exercise pagination, revoked access, account mismatch and currency/timezone rules. No campaign spending is enabled. The interface distinguishes login connected, Ads authorized and reporting healthy.

### T41 — Add Reddit Ads reporting

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T39.
- **Inspect first:** Reuse reporting contracts; current main has no proven Reddit advertising integration.
- **Execute:** First document actual Reddit Ads API access, scopes, quotas and approval requirements. Implement reporting only for supported authorized access, with campaign/creative identities and clear metric definitions. Keep Reddit organic mention monitoring separate from advertising. If access is unavailable, complete the adapter contract/fixtures and expose a blocked capability instead of fake data.
- **Done when:** Supported reports reconcile with fixtures and, when available, an authorized account. An inaccessible API is a recorded external blocker, not a completed integration. The provider is isolated from other tenants and does not silently substitute scraped metrics.

### T42 — Make the idea board feed versioned creative work

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T32,T33.
- **Inspect first:** marketing_documents, Idea board, Media studio and existing text-graphic PNG generation.
- **Execute:** Introduce reusable brief/idea IDs and derived asset versions. Carry audience, offer, hook, brand rules, claim, copy, format, design choices and experiment IDs across destinations. Add one server-side image-generation adapter with preview/edit/retry and tenant usage limits. Keep the current simple graphic tool available where useful.
- **Done when:** One idea creates an email/ad/social draft without retyping the brief. Generated media records prompt/model/asset lineage and can be reviewed. Color/style/copy metadata links to later performance; the system does not infer causality from those tags alone. Provider failure preserves the draft.

### T43 — Add asynchronous video-ad production

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T42.
- **Inspect first:** Media asset/version contract and the shared job system; choose a supported video provider after checking current official capabilities/costs.
- **Execute:** Implement brief → storyboard/script → reviewed generation job → preview → export. Add dimensions/duration/brand constraints, progress, cancellation, retry, asset retention and per-tenant budget limits. Preserve music/voice/image usage metadata where relevant and distinguish generated media from measured campaign performance.
- **Done when:** A test generation or provider fixture runs through all states, including failure and retry, without duplicate billing requests. The output is linked to its source idea and creative version. Generating a video does not publish an ad or spend an advertising budget.

### T44 — Publish blog content to one connected existing website

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T32,T42.
- **Inspect first:** Blog studio drafts, marketing documents, website connections and current absence of a full publishing service.
- **Execute:** Add reviewed publish/update/schedule flows for one documented external CMS/site adapter selected for the first founder. Track canonical URL, slug, metadata, author, status and version. Carry idea/evidence links. Separate draft from published revision; implement safe retry and an update/rollback strategy. Feed search and conversion metrics back to the content record.
- **Done when:** An article publishes to the chosen test site, can be updated without duplicate posts and reports its real URL/status. Connection loss does not mark a draft published. The same content can later use the native CMS from T49.

### T45 — Publish social content through one complete adapter

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T42,T34.
- **Inspect first:** Social Media preview, connection capabilities and asset/publish jobs.
- **Execute:** Choose the first target network for the founder and implement its current official authorization, account/page selection, media constraints, preview, scheduling, publish result and analytics. Separate social publishing permissions from advertising permissions. Reuse the idea and creative records. Treat each additional network as a separate reviewed adapter task using these same acceptance criteria.
- **Done when:** The selected test account receives the reviewed content once, with a provider post ID and status reconciliation. Expired authorization, unsupported media and timeout outcomes are handled. Other networks remain explicitly unavailable until their individual adapters pass, rather than being advertised as supported by association.

### T46 — Define a safe, versioned site/page document model

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T11,T24,T42.
- **Inspect first:** Landing pages/Funnels previews, website form connections and tracker contracts.
- **Execute:** Create tenant-owned sites, pages, components, styles, assets, breakpoints and revisions as structured documents. Define reusable sections, responsive stacks, relative/absolute positioning and validation. Distinguish draft/preview/published versions. Start with a small component set covering a marketing page, form, pricing and booking.
- **Done when:** A saved page reopens with the same structure; invalid or cross-tenant references fail. Undo/version history works. A page revision maps directly to analytics and experiments. Arbitrary generated scripts are not executed as trusted application code.

### T47 — Implement the responsive design canvas

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T46.
- **Inspect first:** Page schema and existing UI components; use a scoped editor rather than rewriting the CRM.
- **Execute:** Build drag/layout editing, responsive stacks, spacing, typography, assets, selection, undo/redo and breakpoint overrides. Support desktop presets such as 1440/1200, tablet/mobile and custom widths. Add absolute/relative positioning with overflow guidance. Provide keyboard editing and a reliable preview.
- **Done when:** A founder can build, save and reopen a landing page across at least desktop/tablet/mobile without clipped content. Responsive overrides do not corrupt the base layout. Editor navigation, large pages and invalid image states are usable and tested.

### T48 — Publish sites with reliable hosting and domains

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T47.
- **Inspect first:** Existing deployment infrastructure and provider domain/DNS capabilities; do not assume the CRM host is already a multi-site publishing platform.
- **Execute:** Implement immutable site builds, preview/publish/rollback, custom-domain ownership verification, TLS status, asset hosting and publish logs. Choose CDN/cache/image/font delivery mechanisms and benchmark them. Separate connecting an existing domain from purchasing a domain. Expose purchase only after registrar eligibility, price, renewal and approval flows are designed and tested.
- **Done when:** A test site publishes, updates and rolls back; domains cannot be claimed across tenants without proof; failed builds leave the last good version live. Performance measurements support claims. No claim of unlimited traffic absorption or a fixed domain price is made without evidence.

### T49 — Add the native visual CMS

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T46,T48.
- **Inspect first:** Native page/asset versions and Blog studio's content model.
- **Execute:** Add collections, typed fields, references, rich text, draft/review/publish states, pagination and dynamic page binding. Support blog posts, team profiles and simple content pages first. Reuse Blog studio drafts and provenance. Define schema-change handling and localization-ready content identities.
- **Done when:** An editor can add a blog entry and publish its dynamic page without code. Broken references and invalid schema changes are caught. Draft content is not exposed publicly. Rollback and content permissions work per tenant.

### T50 — Generate editable websites from AI briefs

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T30,T47,T49.
- **Inspect first:** Business context, design schema, CMS and media generation contracts.
- **Execute:** Add a server-side layout/copy generation tool that produces validated page documents, not opaque HTML pasted into the CRM. Generate a preview with source claims/brand constraints and explain material changes. Support targeted edits and reviewable diffs. Keep continuous-update suggestions as drafts until approved.
- **Done when:** A brief creates a responsive page whose components remain editable. Unsupported components or unsafe content fail validation. AI changes preserve manual work unless the reviewed diff explicitly replaces it. Generating a site never automatically changes a live custom domain.

### T51 — Finish motion, SEO, accessibility and localization

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T47,T48,T49.
- **Inspect first:** Published-site renderer, page metadata, CSS and CMS.
- **Execute:** Add controlled entrance/hover/scroll interactions with reduced-motion support, semantic markup, sitemap/robots/canonical metadata, image alt text, compression and optimization where supported. Add locale routing/content variants and performance budgets. Measure representative pages instead of claiming clean output or GPU acceleration without verification.
- **Done when:** Keyboard/screen-reader checks and mobile overflow checks pass for reference sites. Reduced motion works. Search metadata and locale links match published versions. Core page performance is measured before/after animations and optimized assets.

### T52 — Create and manage reviewed Google Ads campaigns

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T39,T42,T48.
- **Inspect first:** Read-only Ads adapter, campaign/creative versions and action approvals.
- **Execute:** Add campaign/ad drafts, validated destinations, budget ceilings, account selection, preview/approval, provider validation and publish/update/pause. Associate exact creative and landing revisions with an experiment. Use provider idempotency/reconciliation where supported and handle partial failure. Keep automatic budget optimization disabled until separately validated.
- **Done when:** Test fixtures or authorized nonspending validation prove payloads, approvals and partial failures. A production launch requires explicit reviewed account/creative/budget authorization. Editing the budget or creative invalidates the previous approval. Results connect back to the original idea.

### T53 — Add reviewed Meta Ads campaign execution

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T40,T42,T52.
- **Inspect first:** Meta reporting/authorization and shared ad-action approval contracts.
- **Execute:** Implement provider-specific campaign/ad-set/ad creation, placement/media validation, budget limits, review and pause. Preserve provider-specific concepts instead of forcing Google fields into Meta. Add operation logs and recovery from partial campaign creation.
- **Done when:** Mock/test validation covers permission loss, rejected creative, partial success and duplicate requests. Only supported approved assets/accounts can publish. Reporting joins to the actual provider IDs; ad spend cannot start through an unreviewed model action.

### T54 — Add reviewed Reddit Ads campaign execution

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T41,T42,T52.
- **Inspect first:** Verified Reddit Ads capabilities and shared campaign execution contracts.
- **Execute:** Confirm write access and supported operations, then implement reviewed draft/validate/publish/pause with provider-specific budget and creative rules. If write access is unavailable, complete the capability report and keep execution disabled; do not replace it with browser automation or guessed endpoints.
- **Done when:** Supported operations have fixture and authorized provider evidence. A blocked API is clearly marked as unavailable. Approval, budget caps, deduplication and partial-failure recovery are proven before any real campaign can run.

### T55 — Analyze call scripts with evidence and transcript timing

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T13,T25,T31,T33.
- **Inspect first:** Existing call scripts, outcomes and recording-choice/playback paths; add transcript and script-version models.
- **Execute:** Add opt-in transcription, redaction, access/retention and timestamps. Version script sections and record which section a rep reached; align actual spoken text cautiously. Report objections, hang-ups, demos and later trial/payment outcomes by script version/segment/rep. Let the AI draft a testable script variant.
- **Done when:** A result links to an authorized recording/transcript excerpt or a documented rep marker. Denominator includes only calls reaching that section. Unrecorded/unaligned calls are labeled unknown. The AI says where hang-ups are associated, not that a phrase caused them; variants are tested before claiming improvement.

### T56 — Connect SaaS Launchup's own subscriptions and entitlements

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T05,T27.
- **Inspect first:** platform_subscriptions, platform_subscription_events, app/api/operator/route.ts and platform Settings/Billing.
- **Execute:** Add a separate platform billing account/adapter and webhook ledger for founder plans, trials, invoices, cancellation and feature/usage entitlements. Reuse billing infrastructure without sharing tenant-provider credentials or metrics. Record any manual administrative overrides with expiry/reason. Choose pricing from measured usage and business decisions, not the earlier illustrative $30 website figure.
- **Done when:** A founder platform subscription changes only platform access/billing. The founder's customer's SaaS payments remain separate. Failed/late webhooks reconcile; cancellation policy and access end dates are explicit. Operator overrides cannot silently alter tenant MRR.

### T57 — Synchronize Google Calendar with appointments

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T15,T18,T10.
- **Inspect first:** Existing Google Products read-only probes and canonical appointment model.
- **Execute:** Add deliberate calendar selection and required write/free-busy capabilities. Sync booked/rescheduled/cancelled events and external busy changes, store external IDs, and reconcile conflicts/webhook renewal. Define loop prevention and change authority. Add join/conference links only when supported and verified.
- **Done when:** A booking appears once on the selected staging calendar; rescheduling/cancellation updates it; external busy time prevents conflicts. Reconnect and duplicate callbacks do not duplicate events. Reminders use the final canonical time, not a stale provider response.

### T58 — Add Microsoft calendar integration

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T57.
- **Inspect first:** Shared calendar interface; verify current Microsoft identity, Graph scopes and notification requirements.
- **Execute:** Implement account/calendar selection, refresh/reconnect, free-busy, create/update/cancel and incremental synchronization. Preserve Microsoft-specific event/version details. Reuse appointment conflict and reminder behavior while keeping tenant account credentials isolated.
- **Done when:** The Google calendar acceptance cases pass against Microsoft fixtures and a configured test account. Tenant/calendar mismatch and revoked permission fail clearly. Simultaneous local/external changes follow the documented conflict policy.

### T59 — Integrate Calendly without double-booking ownership

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T15,T18,T10.
- **Inspect first:** Canonical appointments and currently absent Calendly adapter.
- **Execute:** Verify the target account's API/webhook capabilities and plan restrictions. Support importing bookings, cancellations and reschedules; generate/manage scheduling links only where supported. Designate Calendly or SaaS Launchup as scheduling authority per booking type and map host/timezone/invitee identities.
- **Done when:** An external booking produces one canonical appointment and the intended reminder policy. A reschedule cannot leave two active bookings or two reminder sets. Unsupported scheduling operations are clearly described rather than promised. Replayed notifications are safe.

### T60 — Build loyalty, education, appreciation and referral workflows

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T21,T28,T34.
- **Inspect first:** Retention/value metrics, feedback, marketing content and automation actions.
- **Execute:** Add education milestones, success check-ins, appreciation/reward records and referral codes with attribution. Identify retention risks from configurable signals and verified context. Track reward eligibility/redemption and cost. Connect recommendations to repeat customer value, expansion and retained revenue.
- **Done when:** A renewal/value cohort can receive a reviewed education workflow; rewards and referral conversions are deduplicated. Incentives are not tied to positive reviews. Inactivity is a risk signal, not automatic churn. Outcomes include customer success and program cost, not just messages sent.

### T61 — Create reusable native journey templates

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T33,T34,T46,T60.
- **Inspect first:** Versioned journey rules, page/form/script/message assets and automation definitions.
- **Execute:** Package reusable funnels, forms, events, metrics, automations and content as a versioned template manifest. Include required integrations, variable mappings, evidence/limitations, dependency validation, preview/diff and rollback strategy. Instantiate into isolated tenants with fresh IDs and secrets supplied separately.
- **Done when:** One template installs into two clean workspaces without sharing customers, credentials, consent, event history or publishing destinations. Unsupported assets are reported. Updates show conflicts and preserve client edits unless a reviewed change replaces them.

### T62 — Assess and implement bounded HighLevel snapshot interoperability

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T61.
- **Inspect first:** Native template manifest and current official HighLevel Snapshot/API documentation [R4].
- **Execute:** Create a capability matrix for what can be inspected/exported/imported through authorized APIs or provided files. HighLevel snapshots move configuration within its ecosystem and exclude live customer records/connections; they are not automatically a native SaaS Launchup runtime. Build only supported mappings, with import preview, unsupported-asset report and credential reconnect steps.
- **Done when:** A sample authorized asset set either imports with validated mappings and fresh IDs or produces a precise compatibility report. Contacts, conversations, appointments and secrets are not assumed to arrive in a snapshot. No promise of universal one-click compatibility without a working fixture.

### T63 — Introduce agency organizations and delegated client access

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T06,T56,T61.
- **Inspect first:** Existing tenants/memberships/operator separation and client workspaces.
- **Execute:** Add agency-to-client relationships, delegated roles, client-owned data and explicit scope switching. Preserve solo-founder workflows. Implement auditable client provisioning, access revocation and transfer/export. Separate agency delegation from platform-operator privileges.
- **Done when:** Agency staff can access only assigned clients; a client cannot inspect sibling clients; revocation applies to API, caches and model retrieval. A solo founder remains one workspace. Client departure preserves ownership and provides a documented transfer/export path.

### T64 — Add white-label presentation and domain provisioning

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T48,T63.
- **Inspect first:** Tenant branding, custom-domain verification, emails, booking pages and auth callback configuration.
- **Execute:** Add agency/client brand configuration for logo, colors, approved domains and customer-facing messages. Verify domain ownership and certificate lifecycle; handle branded login/booking/support links and sender identity deliberately. Keep identity/security boundaries tied to tenant/domain mapping.
- **Done when:** Two brands render independently with no shared assets or data leaks. A custom domain cannot be claimed by another agency. Login callbacks, booking links and transactional messages reach the correct workspace. Removing branding does not destroy client data.

### T65 — Finish agency operations, reporting and unit economics

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** T60,T63,T64.
- **Inspect first:** Agency roles, metric snapshots, template rollout and platform billing.
- **Execute:** Add permitted cross-client summaries, delegated approval queues, template rollout history, usage/cost limits and agency/client billing relationships. Distinguish aggregated reports from client data exports. Show what an agency may approve for a client and record each delegation. Do not share raw tenant data for cross-customer model learning by default.
- **Done when:** Agency totals reconcile to authorized clients without double-counting accounts/currencies. A client-specific limit or provider failure does not stop other clients. Approvals and charge attribution are auditable; account offboarding revokes access and credentials under the agreed policy.

### T66 — Run the full release audit and controlled launch

- **Status:** Not started / not yet verified by this plan.
- **Depends on:** All intended release capabilities above; do not claim optional or blocked modules as complete.
- **Inspect first:** docs/implementation-status.md, task evidence, staging deployment, monitoring, data retention and rollback procedures.
- **Execute:** Run the final journeys listed in section 9 with two tenants and multiple roles. Review security, accessibility, performance, event/LLM/media/delivery costs, quota enforcement, support runbooks, provider outages, export/deletion and backup restore. Reconcile production release SHA, migrations and feature flags. Publish only the feature set that passed, with a small monitored rollout and rollback plan.
- **Done when:** Every advertised workflow has repeatable evidence and an owner for operational failures. Dashboards reconcile with sources, the AI passes its evaluation gate, provider side effects are controlled, and a rollback/restore rehearsal exists. Blocked features remain disabled and accurately labeled.

## 8. Copy-and-paste Codex prompts

### First prompt — execute T00 only

```text
Read docs/SAASLAUNCHUP_EXECUTION_PLAN.md and all applicable repository instructions.
Execute T00 only. Do not implement the full roadmap.

The plan audited main at 13940c30651e81d682ca181f4975d232245f37ba.
First record the actual current SHA and inspect any changes since that baseline.
Cross-reference current files before treating an audit finding as still open.
Preserve current customer data and existing working features.

Establish the actual deployment, production/staging separation, applied migrations,
current authentication model, provider capabilities and authenticated menu inventory.
Run the existing local checks, using isolated local/staging data only.
Do not expose secrets or contact real prospects, send messages, place calls,
buy domains, change ad spend or alter production data as part of this audit.

Create/update docs/implementation-status.md and docs/evidence/T00.md with:
- current repository/deployed SHA evidence or the specific unresolved gap;
- status for each menu and provider capability;
- exact commands and observed results;
- integration/account prerequisites without secret values;
- the first executable next task and any changed assumptions.

Finish the accessible work even if one external account is unavailable.
Mark blocked checks precisely; do not claim they passed.
Return changed files, findings, validation and blockers. Stop after T00.
```

### Subsequent implementation prompt

```text
Read docs/SAASLAUNCHUP_EXECUTION_PLAN.md, current implementation status,
applicable repository instructions and dependency-task evidence.
Execute TXX only.

Inspect the current implementation first. Reuse working code and prove any
already-complete acceptance criteria. Implement only the remaining scope.
If the repository has changed, explain the relevant difference and adapt this task.
Do not restore obsolete routes, migrations or assumptions from older plans.

Add meaningful verification for the risks introduced by this task.
Use isolated local/staging data and mock external side effects by default.
Update docs/implementation-status.md and docs/evidence/TXX.md.
Return the behavior change, files, test evidence, limitations and migration/rollback
notes. Prepare one reviewable PR if repository access supports it.
Stop after this task. Do not execute the next ID automatically.
```

Replace `TXX` with the selected ID, such as `T01`.

### CodeRabbit review brief

```text
Review this PR against task TXX in docs/SAASLAUNCHUP_EXECUTION_PLAN.md.
Check the actual diff and acceptance evidence, not just the PR summary.
Focus on tenant/role isolation, identity trust, data preservation, migrations,
idempotency, concurrent updates, provider failure, consent/suppression where
applicable, and whether the UI claims more than the backend proves.
For analytics/AI tasks, check denominators, cohort windows, provenance,
uncertainty, unsupported causal claims and cross-tenant evidence access.
Identify concrete correctness or security issues with file references and impact.
Keep unrelated refactors out of this task.
```

### Required task evidence template

```markdown
# TXX evidence

- Status: pending / in progress / implemented / verified / blocked
- Started from commit:
- Finished commit or PR:
- Environment and deployment reference:
- Existing implementation reused:
- Behavior changed:
- Data/schema changes and migration path:
- Acceptance criteria checked:
- Commands and actual results:
- Runtime/UI/API proof (redacted):
- Tenant/role and failure cases exercised:
- External prerequisites still missing:
- Known limitations:
- Rollback or recovery:
- Reviewer findings and resolution:
- Next eligible task:
```

`Implemented` means code exists. `Verified` means the task's acceptance evidence exists in the stated environment. A locally verified task may still need a production release check. A provider account that has not been configured is an external dependency, not proof of a broken adapter or a completed integration.

## 9. End-to-end acceptance scenarios

Use synthetic fixtures. Counts below are test expectations, not observations about the current business.

| Scenario | Required evidence |
| --- | --- |
| Founder onboarding | A new non-hard-coded approved test founder creates exactly one workspace, can return after logout, and has no access to the second test tenant. |
| Form → lead → demo | A versioned test form creates one CRM contact/opportunity/follow-up; retry deduplicates; the correct booking page opens; the appointment persists. |
| Cold call → booked demo | A permitted test call records outcome/script version; a reviewed booking link reaches only the designated test recipient; booking updates the timeline. |
| Reminder schedule | A booking more than 24 hours away produces seven intended channel jobs: 2 immediate, 2 at −24h, 2 at −4h, 1 at −15m. Advance a test clock rather than waiting a day. |
| Reminder interruption | Opt-out, cancellation, reschedule and provider timeout each produce the expected skipped/revised/reconciled jobs without stale or duplicate outreach. |
| Trial analytics | A controlled cohort of 10 eligible trial accounts, 4 activated and 2 qualifying paid conversions shows 40% activation and 20% conversion under the stated completed window; repeat events do not change totals. These fixtures test math, not statistical significance. |
| Checkout and cancellation | Failed, pending and successful payment states differ. A scheduled cancellation differs from effective churn. A support cancellation ticket alone changes neither. |
| MRR and retention | Annual/monthly plan normalization, expansion, contraction, churn, reactivation and currency grouping match an independently calculated ledger. |
| Experience across stages | A pretrial question, trial bug and paid support case appear in Experience with their original stage/account context; a linked improvement task closes the loop. |
| Grounded AI | Every quantitative claim cites a permitted metric/evidence record; missing or contradictory data produces an honest limitation; customer-provided text cannot trigger an unauthorized action. |
| Reviewed improvement | A recommendation produces an editable draft; approval binds its exact version; exposures and outcomes link back to it. An inconclusive experiment remains inconclusive. |
| Publishing | A reviewed page/blog/social/ad asset reaches the selected test destination once, with a provider result, error recovery and applicable rollback/pause. |
| Privacy and lifecycle | Retention, customer export/deletion and credential revocation propagate through events, derivatives and retrieval indexes under the documented policy without leaking other tenants. |
| Agency isolation | Agency members see only delegated clients; client brands/domains remain isolated; revocation and offboarding work without destroying client data. |
| Operational recovery | Queue/provider outage, failed deployment and database restore have rehearsed, documented recovery. Monitors identify missing data and failed delivery before a dashboard silently misleads a founder. |

## 10. Product decisions already supplied, defaults, and real dependencies

These are not a new questionnaire. Agents should use the existing decisions and record defaults rather than stopping for routine implementation choices.

| Item | Direction |
| --- | --- |
| First buyer | Solo SaaS founder. Agency/white label is phase two. |
| Dashboard | Awareness, Consideration, Purchase, Experience, Loyalty with advocacy inside Loyalty initially. |
| Consideration | Trial/evaluation and in-product onboarding/use, according to a configurable rule. |
| First workflows | Form → booking; trial → activation → paid; support → improvement task; called → demo → exact confirmation/reminder chain. |
| Trial length and activation | Tenant-specific, configured on product setup. No universal forced 14-day trial; the existing booking invite expiry is unrelated. |
| Product value | Founder defines one primary first-value event and repeat-value events. Usage duration alone is not proof of success. |
| Attribution window | Configurable, explicit and versioned; report unknown until configured or show a clearly labeled default. |
| Billing | Stripe is the first tenant SaaS adapter because some connection work exists; other providers use the same normalized contract later. Platform billing is separate. |
| Email provider | T16 selects/configures one transactional sender; do not infer that an existing Gmail read connection can deliver campaigns. |
| AI | Model gateway + curated playbooks + tenant context + deterministic metrics + evaluation; reviewed actions before sends/publishing/spend. |
| GHL snapshots | Compatibility investigation and bounded mappings, plus native versioned templates. No universal import claim. |
| Builder pricing/domain sales | Later commercial decision backed by actual hosting, generation, registrar and support costs. The earlier example price was a concept, not a verified current offer. |
| External dependencies | Approved staging login, deployment metadata, selected provider test accounts/scopes, sender/domain verification and provider approvals. Record each as configured, absent or unverified. |

## 11. References and evidence boundaries

Repository evidence is pinned to [the audited commit](https://github.com/SharpConceptions/SaasLaunchUp/tree/13940c30651e81d682ca181f4975d232245f37ba). Referenced source paths throughout this document identify where the findings came from. External documentation below supports specific integration constraints; it does not establish that those integrations are deployed or connected in SaaS Launchup.

- **R1 — Google Ads authorization:** the official REST documentation requires OAuth and a developer token, with applicable manager-account context. Used for F08 and T10. [Official documentation][R1]
- **R2 — Stripe subscription events:** the official subscription webhook guide supports using lifecycle/payment events and reconciliation rather than manual purchase labels as billing truth. Used for T27. [Official documentation][R2]
- **R3 — Search Console Search Analytics:** authorized property queries provide search traffic dimensions/metrics. This is distinct from universal internet rankings or AI-answer citations. Used for T37. [Official documentation][R3]
- **R4 — HighLevel snapshots:** the official overview describes reusable configuration assets and excludes contacts, appointments, conversations, live activity and third-party connections. Native SaaS Launchup interoperability requires a separately proven mapping. Used for T62. [Official documentation][R4]

[R1]: https://developers.google.com/google-ads/api/rest/auth
[R2]: https://docs.stripe.com/billing/subscriptions/webhooks
[R3]: https://developers.google.com/webmaster-tools/v1/searchanalytics/query
[R4]: https://help.gohighlevel.com/support/solutions/articles/48000982511-snapshots-overview

**What this audit does not establish:** private production-screen behavior, actual provider secrets/account readiness, deployed SHA, live delivery success, real billing correctness, provider approval, penetration-test coverage or production upgrade/restore safety. Those items have explicit tasks and release gates above. The plan is complete as a handoff; the application becomes complete only when the corresponding evidence is recorded.
