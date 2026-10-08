# Architecture and engineering decisions

## Goal
Nationwide map-first investment discovery for US single-family homes and duplexes, initially proprietary then multi-tenant SaaS with CRM and controlled automation. Core calculations deterministic. No model may claim nationwide complete active listings without licensed feed coverage.

## Stack
Next.js/TypeScript/Tailwind/MapLibre; Python/FastAPI; PostgreSQL/PostGIS; Polars/DuckDB ETL; scikit-learn and LightGBM; Postgres-backed job queue first; S3-compatible blobs; Docker Compose; Stripe; Gmail API/Microsoft Graph/IMAP-SMTP where supported. Optional LLM adapter only for language work.

## Logical services
Ingestion -> immutable raw snapshots -> normalized property identity + coverage metadata -> comps and point-in-time feature store -> AVM with calibrated intervals -> deterministic underwriting -> ranked map queries -> CRM/communications -> approval-gated agents -> transaction feedback.
API routes: /v1/search, /v1/properties/{id}, /v1/properties/{id}/comps, /v1/valuations/{id}, /v1/deals, /v1/contacts, /v1/mailboxes, /v1/tasks, /v1/coverage. Use typed OpenAPI contracts.

## Data rights
Free: Census ACS/TIGER/geocoder, FHFA HPI, FRED, FEMA flood, selected county assessor/recorder and municipal datasets, OpenStreetMap with attribution and compliant usage. Active listings: RESO/MLS or commercial feeds with explicit licenses; never assume public access, reuse, or completeness. Store source URL, retrieval time, licensing terms, field-level provenance, refresh SLA, allowed redistribution, and geography coverage. County data varies significantly.

## Modeling
Build comps baseline first; deterministic filters for arm's-length recent sales and similarity; LightGBM challenger. Train geographically aware, time-split models; separate duplex features/rental economics. Persist model version, inputs, selected comps, adjusted prices, prediction interval, reason codes. Benchmark MAE/MdAPE, coverage, realized net return after friction, time-to-sale, and precision at top-K. Guard against target leakage, non-arm's-length transfers, condition gaps and selection bias. LLMs cannot set prices or ROI.

## Underwriting
Purchase + closing + title + repair contingency + taxes + insurance + carrying + financing if any + brokerage/selling + transfer tax + concessions. Resale price scenarios and time-to-exit. Calculate net profit, ROI, annualized return, downside and max offer. Every assumption versioned, locality-sensitive and inspectable. Cash deals default, financing optional.

## CRM
Organization-scoped contacts, property relationships, deals, stages, notes, tasks, buyer buy boxes, mailbox accounts, message threads, documents, communication consent and audit. Gmail OAuth, Microsoft Graph, Yahoo and generic IMAP/SMTP where supported. Encrypt secrets, minimum scopes, tenant isolation. Drafts can be automated; sending and binding offers require approval.

## Cheap operations
One small server, Postgres/PostGIS, object storage; batch and incremental ingestion, deduplication, indexed bounding-box queries, cached features and valuations, no LLM on map pan, queued jobs, per-tenant quotas. Avoid Kubernetes, microservices and paid vector DB. Costs scale with geographic coverage and licensed data.

## Security and law
Tenant row-level isolation, audit, rate limiting, encryption, backups, OAuth revocation. Human authorization for communications and contractual/fund actions. Respect CAN-SPAM/TCPA, fair housing, state brokerage and wholesaling laws, real estate data licensing. Never silently contact owners or buyers.

## References
https://github.com/maplibre/maplibre-gl-js
https://github.com/openmaptiles/openmaptiles
https://github.com/fastapi/fastapi
https://github.com/postgis/postgis
https://github.com/duckdb/duckdb
https://github.com/pola-rs/polars
https://github.com/microsoft/LightGBM
https://github.com/dmlc/xgboost
https://github.com/RESOStandards
https://github.com/langchain-ai/langgraph
https://github.com/nextauthjs/next-auth
https://github.com/stripe/stripe-node
