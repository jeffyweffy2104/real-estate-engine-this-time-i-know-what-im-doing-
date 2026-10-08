# Distressed Acquisition Intelligence — Executable Engineering Plan
Version 1.0 | Maryland-first, nationwide adapters | 2026-10-07

## Mission and acceptance
Extend the Real Estate Engine with a reproducible, low-cost, mostly deterministic acquisition pipeline for single-family homes and duplexes: foreclosure, pre-foreclosure, REO, probate/estate listings, tax sale, city-owned property, stale listings and price cuts. Search anywhere in the US by map/search; Maryland is the first live jurisdiction. Surface *net realizable profit*, cash-to-close, risk, provenance, and actionable CRM workflows. Never confuse appraised value with sale proceeds. No unattended offers or owner outreach.

## Product boundaries
- Deal sourcing, valuation, underwriting, alerts, CRM, source links, manual verification, lender comparison, scenario simulation.
- No fabricated listing access; MLS/RESO and proprietary sites require agreements. Do not bypass CAPTCHA, paywalls, robots or access controls. Do not scrape Maryland SDAT's interactive real-property search (expressly prohibited); prefer Maryland open-data parcel feeds with checked licensing and freshness.
- Foreclosure Registration System is restricted; public aggregate foreclosure tracker is *not* a parcel-level lead feed. Use lawful courthouse, trustee, publication, county, or licensed data.
- Tax sale certificates are NOT deeds; model redemption, priority, legal fees and timelines separately. Probate filing is NOT proof of a sale or a representative's authority. Mark unverified leads as research-only.
- Fair housing, privacy, CAN-SPAM/TCPA, do-not-call, Maryland solicitation restrictions, provider terms, data retention, access audit, opt-out and suppression checks before CRM outreach. No automatic outreach to bereaved persons.
- FHA 90-day resale rule and potential additional appraisal in days 91–180; eligibility flags, exceptions and versioned legal rules must be reviewed by closing counsel.

## Suggested architecture
Monorepo: apps/web (Next.js, map/search), apps/api (FastAPI or TypeScript service), packages/core (pure deterministic calculations), packages/db (Postgres/PostGIS migrations), packages/connectors (isolated source adapters), packages/workers (cron jobs), packages/crm (email OAuth + communications), packages/ui (shared components), infra/ (deployment and CI), docs/.
Low-cost deployment: Vercel web; Neon Postgres/PostGIS if supported in chosen tier; scheduled GitHub Actions/Cloud cron for small ingest, durable queue for growth; S3-compatible object storage for permitted source snapshots; MapLibre + OpenStreetMap-compatible licensed tiles; avoid commercial geocoding until needed. Track per-source costs. No LLM in core scoring; optional AI only for document extraction with human verification.

## Data model (migration targets)
properties(id UUID, canonical_address, unit, lat, lon, county_fips, parcel_id, property_type, beds, baths, sqft, year_built, owner_occupancy_unknown, geometry, created_at)
source_records(id, source_id, external_id, source_url, observed_at, effective_at, raw_checksum, raw_location, rights_status, confidence, parser_version)
property_aliases(property_id, normalized_address, jurisdiction, parcel_id, match_confidence, evidence)
ownership_events(id, property_id, event_type, recorded_at, party_reference, source_record_id, verification_status)
distress_events(id, property_id, type, stage, event_date, auction_date, court_case_ref, status, evidence_id, last_verified_at)
listings(id, property_id, channel, listing_id, price, status, first_seen, last_seen, price_history_json, source_record_id)
sales(id, property_id, closed_at, consideration, arms_length_flag, source_record_id)
comps(id, subject_property_id, comparable_property_id, distance_m, recency_days, adjustment_json, weight, excluded_reason)
valuations(id, property_id, as_of, estimate, low, high, comp_count, methodology_version, confidence, provenance_json)
deal_scenarios(id, property_id, purchase_price, rehab, contingency, lender_id, financing_json, holding_days, resale_low, resale_base, resale_high, taxes_fees_json, cash_required, expected_profit, downside_profit, irr, roi_on_cash, status)
contacts(id, entity_type, name, role, lawful_source, verified_authority, opt_out_at)
crm_deals(id, property_id, stage, assigned_to, next_action_at, scenario_id)
communications(id, deal_id, provider, external_thread_id, direction, sent_at, consent_basis, status)
alerts(id, property_id, rule_id, triggered_at, delivered_at, dedupe_key)
audit_logs(id, actor_id, action, resource_id, timestamp, before_after_hash)
All records scoped by tenant_id in multi-user deployment. Encrypt tokens and sensitive contact data; row-level authorization. Unique (source_id, external_id); dedupe with parcel+county first, address second, fuzzy only with review.

## Data adapters: Maryland first
1. Maryland Open Data real property assessment datasets and Maryland Planning parcel/geospatial downloads; verify exact dataset, field names, update cadence, licensing, owner-data restrictions before coding.
2. County/city open data for code violations, vacant housing, city-owned inventory, permits, tax assessments and published sales.
3. Maryland Judiciary public case search, Registers of Wills, court foreclosure notices, trustees' published auction notices: check terms, access methods, pagination and machine-use permissions; otherwise build manual CSV/import workflows.
4. HUD HomeStore/HUD Home listings via authorized sources; licensed REO feeds and brokerage data.
5. MLS/RESO Web API only with broker/license agreements. Without it, user-imported CSV and links are MVP.
6. Nationwide connector interface with capabilities, geography, licensing, pagination, rate limits, incremental cursors, and freshness; disabled until authorized.
Source registry fields: jurisdiction, license, access_method, allowed_storage, allowed_display, allowed_outreach, rate_limit, cost, last_reviewed, enabled. Every event links to evidence; retain source timestamp and retrieval timestamp separately.

## Deterministic deal engine
Comp selection: closed arms-length sales, same property class, recent 3–6 months initially, nearby radius adaptive, sqft/beds/baths/condition adjustments, robust median, explicit outlier exclusions. Low confidence with <3 credible comps; no unsupported ARV.
Cost stack: purchase, acquisition tax/transfer/recordation/title, loan points/origination, interest by days, property taxes, insurance, utilities, repairs, permits, contingencies, commissions, seller concessions, disposition closing costs, lien/payoff estimate, legal/title, occupancy/eviction risk.
Profit = realized resale - purchase - all purchase/rehab/carry/sale costs. Cash required = equity/down payment + nonfinanced closing + rehab cash timing + reserves. Capital recycled at closing = net proceeds after lender payoff, taxes/fees and outstanding liabilities. Model LTV and LTC independently; enforce lender min-loan, seasoning, prepayment and draw schedule.
Output three scenarios (bear/base/bull), maximum allowable offer (MAO), break-even resale, holding-period sensitivity, capital-at-risk, expected profit and annualized return; downside constraints before rank.
Score = weighted verified discount + downside-adjusted net profit/cash + liquidity/sell-through + data confidence - title/condition/occupancy/financing/seasoning penalties. Publish weights and explain score; exclude unknowns rather than pretend certainty.
Default gate: property type SFR/duplex, cash-to-close <= configurable budget (default $50k), verified net base profit >= configurable threshold, bear-case loss limit, minimum comps, current evidence, no unresolved legal title blockers. Flag auction properties for enhanced due diligence.

## UI and workflows
Map/search with geocoded region/address, pins by opportunity type, clustering, filters for event stage, price, estimated value, profit, cash required, source freshness, condition, risk, days on market, auction date. Detail drawer: evidence timeline, property facts, comp map, ARV range, offer calculator, financing assumptions, red flags, source links, CRM action.
Saved searches + daily digest; alerts on price drops, new estate listing, auction schedule change, verified REO listing. Deal board: discovered > screened > diligence > contacted > offer > under contract > rehab/listed > closed/lost. Gmail OAuth first; other mail providers via secure OAuth/IMAP as feasible; never store mailbox passwords. Manual approval for every outbound message, templates with opt-out and audit history.

## Delivery plan: 14 sequential implementation PRs
PR01 Foundation: architecture ADRs, README, pnpm/python workspace, Docker dev, env.example, CI lint/test, seed fixture. Accept: clean clone starts web+API+DB.
PR02 Schema: Postgres/PostGIS migrations, tenant isolation, source provenance, event lifecycle, seed/rollback. Accept: migration round-trip + permission tests.
PR03 Connector framework: typed adapter interface, source rights registry, retry/backoff, cursors, checksums, idempotent ingest, dead-letter queue. Accept: fixture replay identical output.
PR04 Maryland property base: permitted open-data adapter, parcel joins, geometry, metadata freshness, import fallback. Accept: seeded Maryland map with citations and no SDAT site scraping.
PR05 Distress adapters: foreclosure/REO/auction notices via permitted feeds or CSV, stage machine, event deduplication and expiration. Accept: fixture auction reschedule reflected and notified once.
PR06 Probate/estate adapters: lawful record imports, distinguish estate event vs listing, verify representative authority, contact suppression. Accept: probate record cannot trigger unreviewed outreach.
PR07 Tax sale/municipal: certificate vs deed models, redemption and title blockers, city inventory import, eligibility tags. Accept: certificates cannot be modeled as immediately saleable fee-simple houses.
PR08 Comps/AVM: transparent comp retrieval, filters, adjustments, confidence bounds, valuation history. Accept: deterministic golden-fixture results, sparse-comp warning.
PR09 Underwriting: scenario cost waterfall, MAO, cash-to-close, ROI, capital recycling, FHA resale timing warning, sensitivity grid. Accept: golden fixtures reconcile to ledger within $1.
PR10 Opportunity ranking: configurable constraints, reasons, risk flags, saved searches, alert dedupe. Accept: stale evidence/unknown title never labeled investment-ready.
PR11 Map UI: nationwide search, Maryland coverage, layers, cluster pins, detail drawer, mobile responsiveness, accessible keyboard navigation. Accept: e2e search-filter-detail works.
PR12 CRM/email: Gmail OAuth, contact/deal models, threads, approved outbound only, suppression, activity log; Yahoo/IMAP adapter deferred behind provider review. Accept: no message sent without explicit click and permission.
PR13 Operations: scheduled ingest, observability, source health, provider-cost dashboard, secrets, backups, RBAC, retention, rate limiting, alerts. Accept: simulated adapter outage recoverable with no duplicate events.
PR14 Production release: staging deploy, E2E, security checks, user manual, data rights matrix, 50 manually audited Maryland leads, financial model calibration, feature flags and release checklist. Accept: no critical failures, signed-off source permissions and independently reviewed 10 example valuations.

Each PR must include: changed-file manifest, migration plan, tests, acceptance checklist, screenshots if UI, sample fixtures, deployment impact, and follow-up work. Coding agent: implement in order, run tests and linters, open reviewable PRs, never mark acceptance complete without evidence. Avoid synthetic data in production except clearly labeled demos.

## Tests and quality gates
Unit: parcel matching, comp exclusions, negative equity, variable holding time, zero-comps, multi-unit filters, loan points, tax and cost stacks, stale evidence, redemption timelines.
Integration: ingest idempotency, provider failure and rate limits, geospatial bounding, OAuth token refresh, tenant isolation, suppression checks.
E2E: map search > lead > verify evidence > underwrite > CRM > manually approved outreach > disposition.
Performance targets: 95th-percentile map query <2s on Maryland test dataset, 100k+ parcels indexed, 99% successful scheduled ingest excluding upstream outages, alerts idempotent.
Security: encryption, audit trails, minimum privilege, deletion/export, PII minimization, backup restoration drill.

## Reference implementations to study (not copy without license review)
MapLibre GL JS, OpenStreetMap, PostGIS, Martin tile server, OpenAddresses, US Census Geocoder, RESO Web API standards, Airbyte connectors, Temporal workflows, FastAPI, Next.js, Prisma/Drizzle, Supabase Auth or Auth.js, OpenTelemetry, Playwright, Vitest, pytest.
Official source starting points: https://opendata.maryland.gov/stories/s/Maryland-Real-Property-Resources/x8p6-5dqp/ ; https://sdat.dat.maryland.gov/realproperty/pages/default.aspx ; https://www.dllr.state.md.us/finance/consumers/frforeclosuredatatracker.shtml ; https://entp.hud.gov/sfohlp/f17albkhlp-2.6.cfm .
