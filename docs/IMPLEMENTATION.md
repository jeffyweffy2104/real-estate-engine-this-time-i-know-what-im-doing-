# Implementation roadmap

PRs are staged as draft specification PRs. Coding agent must implement code, migrations, tests and documentation before marking ready. Each PR targets main and can be implemented independently with preceding interfaces mocked; merge in numeric order.

## PR 01: Foundation
Branch: spec/01-foundation
Monorepo with apps/web Next.js, services/api FastAPI, packages/contracts; Docker Compose postgres/postgis, make targets, lint/typecheck/pytest/vitest, GitHub CI, env examples, secret scanning. Acceptance: one command boots API/web/db; CI green.

## PR 02: Property database
Branch: spec/02-schema
SQL migrations for properties, parcels, addresses, listing_snapshots, transactions, sources, source_licenses, coverage, valuations, model_versions, comps, organizations, memberships. Stable property identity via parcel+county FIPS, address fallback and merge audit. Spatial GIST indexes; tenant RLS for private tables. Tests duplicate ingest, spatial lookup, migrations.

## PR 03: Data ingestion
Branch: spec/03-ingestion
Adapters for Census geocoder/ACS/TIGER, FHFA HPI, FRED, FEMA, county assessor/recorder open data and licensed RESO feeds. Raw immutable provenance, freshness, usage rights, retry/backoff, idempotent upsert, coverage dashboard. No unauthorized scraping; no nationwide active-listing completeness claim. Fixture tests and source contracts.

## PR 04: Nationwide map search
Branch: spec/04-map
Next.js MapLibre search by city/ZIP/county/address/bounds, PostGIS viewport queries, clustered pins, filters SFR/duplex, price, discount, confidence, cursor pagination and coverage badges. Cache tiles/search; accessible mobile UI. E2E search/filters/empty coverage.

## PR 05: Comparable sales engine
Branch: spec/05-comps
Deterministic same-property-type filters, arms-length recent closed sales, geo radius, sqft/beds/baths/age/condition similarity, outlier rejection, documented adjustment rules and weighted median. Persist chosen comps and reasons; fallback low-confidence. Unit tests synthetic comps and edge cases.

## PR 06: Valuation models
Branch: spec/06-avm
Baseline hedonic regression + LightGBM; train national/regional hierarchical features and separate duplex model; compare to comps baseline; time-split calibration, prediction intervals, feature attribution, model registry and batch inference. No LLM for numeric pricing. Tests reproducibility, leakage, drift.

## PR 07: Historical validation
Branch: spec/07-backtest
Walk-forward evaluation with point-in-time listing/sale features, no future leakage, holdout by geography/time; MAE, MdAPE, interval coverage, top-decile deal precision, realized cost-adjusted profit and liquidity calibration. Reports stratified by county/property type/price and missingness. CI fixture benchmark.

## PR 08: Investment underwriting
Branch: spec/08-underwrite
Deterministic purchase closing, taxes, insurance, utilities, repairs, selling commission, transfer fees, holding costs, capital cost and contingencies; resale scenarios, break-even and max offer; downside liquidity haircut, cash return and IRR. Assumptions versioned by locality. Property-based financial tests.

## PR 09: Property intelligence UI
Branch: spec/09-property-ui
Property details with photos only if licensed, comp map, confidence ranges, price history, ROI scenarios, provenance, investment memo grounded in calculations, exports and watchlists. Explicit stale/coverage warnings. Screenshot/e2e tests.

## PR 10: SaaS tenancy and security
Branch: spec/10-auth
Org/user/roles, RBAC, Postgres row-level security, auth provider abstraction, encryption of credentials, audit log, GDPR/CCPA data requests, per-org usage metering. Cross-tenant access negative tests.

## PR 11: Native CRM
Branch: spec/11-crm
Contacts, organizations, property-contact links, deals, stages, notes, tasks, activity, buyer criteria, seller/agent profiles, custom fields and import/export. State machine for outreach/offer/diligence/closed/resale. Tenant isolation and dedup tests.

## PR 12: Email integrations
Branch: spec/12-email
Gmail OAuth API, Microsoft Graph, Yahoo/generic IMAP-SMTP where supported; mailbox adapter, secure tokens, incremental sync, threading, contact/deal matching, drafts, opt-out tracking, bounce handling and provider rate limits. User confirmation before send, unsubscribe compliance and audit. Mock provider integration tests.

## PR 13: AI workflow agents
Branch: spec/13-agents
Optional budgeted LLM adapter for evidence-grounded memos, inbox summaries, draft responses and document extraction. Deterministic state machine, scoped tools, no direct SQL access, human approval for sending, offers, transfers or contracts. Prompt-injection defenses, citations, audit and cost caps. Offline mock tests.

## PR 14: Buyer matching and resale
Branch: spec/14-disposition
Buyer buy-box profiles, geographic/property criteria, deterministic matching/ranking, buyer outreach drafts, marketing tasks, offers and resale pipeline. Compliance checks for brokerage licensing, wholesaling rules, consent, fair housing and state-specific practices. No automated binding actions.

## PR 15: Subscriptions and quotas
Branch: spec/15-billing
Stripe metered subscriptions and entitlements, free/dev quotas, model runs, data-source rights per plan, usage budgets, tenant spend caps, invoice webhooks and reconciliation. Tests overage, cancellation, webhook idempotency.

## PR 16: Deployment and launch
Branch: spec/16-operations
Single-node Docker deployment, managed Postgres optional, object storage, backup/restore drills, cron incremental ingestion, logs/traces, error budgets, dependency/security scans, disaster recovery, load tests, launch checklist and operator runbooks. Infrastructure target low-volume $20-120/month excluding licensed data.
