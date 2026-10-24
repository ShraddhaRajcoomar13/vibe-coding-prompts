Architecture Summary
Core function (single most important): Allow a young user with no/zero formal credit history to create an authenticated profile, record verifiable positive financial behaviours (on-time “payments”, savings actions, completed financial-literacy modules), accumulate a transparent credibility score + evidence trail, and generate a shareable report that demonstrates responsibility. This is the only path that must work end-to-end for the demo.
Core user journey (happy path):
	1	User signs up / logs in (email + password or magic link).
	2	Completes minimal profile (age band, student status, consent).
	3	Logs or completes a positive action (e.g. “paid mock rent on time”, “saved R200”, “completed budgeting module”).
	4	System records the action with timestamp, evidence reference, and score impact.
	5	User views current credibility score + history.
	6	User generates and downloads/exports a simple PDF or JSON credibility report.
Everything else (real credit-bureau pulls, lender matching, social features, mobile apps, AI coaching, real money movement) is deferred.
Major components:
	•	Lightweight SPA frontend
	•	Single Python backend (FastAPI) exposing REST API
	•	Managed Postgres for durable records
	•	Auth service (managed)
	•	Object storage only if report PDFs are generated
	•	No queues, no microservices, no AI agents in MVP
Key design decisions:
	•	Monolith + clear module boundaries (not microservices) because of 24 h / 1 person constraint.
	•	Score calculation is pure deterministic business logic (easy to test and explain to judges).
	•	All mutations are authenticated and authorised server-side; client never trusted.
	•	POPIA-aware data minimisation from day one.
	•	AWS-centric for easy future production path while staying free-tier friendly for the hackathon.
Assumptions (explicit):
	•	No real money movement or real credit-bureau API calls in the 24 h window (regulatory and integration risk too high).
	•	“Evidence” is self-reported + timestamped for MVP; later can attach receipts or bank-feed proofs.
	•	Target users are 16–25, South African students; age verification is self-declared + parental consent flag for under-18.
	•	Demo can run with seed data and a few real users created during the presentation.

Technology Stack
Layer
Recommendation
Justification
Frontend
React + Vite + TypeScript + Tailwind
Fast to scaffold, excellent DX, component model maps cleanly to the simple screens (login, dashboard, log action, report). TypeScript catches many errors early.
Backend/API
FastAPI (Python 3.12) + Pydantic v2
Native Python, automatic OpenAPI, excellent validation, async-ready, tiny memory footprint. Matches the single-developer Python constraint.
Database
PostgreSQL (AWS RDS free tier or local Docker for demo)
Relational model fits users / actions / scores perfectly; strong constraints and ACID for score integrity.
Authentication
Auth0 (free tier) or AWS Cognito
Managed identity, JWT, social logins later, MFA path. Avoids rolling own password hashing under time pressure. Cognito preferred for pure-AWS story.
External APIs/AI
None for core MVP
Score is deterministic. Later: optional OCR for receipts or simple LLM for report narrative (still behind feature flag).
Testing
pytest + httpx + pytest-asyncio + coverage
Fast, native to FastAPI ecosystem.
Deployment
Frontend: AWS S3 + CloudFront
Backend: AWS App Runner or Elastic Beanstalk (or even a single EC2/t4g.micro for pure speed)
DB: RDS Postgres
Minimal ops, HTTPS by default, free-tier capable. App Runner is simplest for a containerised FastAPI app.
Monitoring
CloudWatch + basic structured logging (structlog)
Free with AWS; enough for demo + early production signals.
Cost / lock-in notes:
	•	All chosen services have generous free tiers sufficient for a hackathon demo and first 100–200 users.
	•	Cognito + RDS + App Runner + S3 stay inside AWS → low lock-in risk for later expansion.
	•	Data residency: choose af-south-1 (Cape Town) if available for POPIA comfort; otherwise eu-west-1 with explicit transfer documentation.
Prototype vs production:
	•	Demo: local Postgres or single RDS instance, Cognito free tier, App Runner or even uvicorn on a t4g.micro.
	•	Production later: multi-AZ RDS, proper secret manager, WAF, stricter rate limits, formal DPIA.

Mermaid Architecture Diagram
flowchart TB
    subgraph TrustBoundary["Trust Boundary - User Device"]
        User[User / Browser]
        FE[React SPA
Vite + TS + Tailwind]
    end

    subgraph AuthBoundary["Auth Boundary"]
        Cognito[AWS Cognito
User Pool + App Client]
    end

    subgraph AppBoundary["Application Trust Boundary - AWS"]
        API[FastAPI Backend
App Runner / EB]
        BL[Business Logic
Score Engine + Validation]
        DB[(PostgreSQL
RDS)]
        S3[S3 Bucket
Reports - optional]
        Logs[CloudWatch Logs
+ Metrics]
    end

    User -->|HTTPS| FE
    FE -->|JWT + REST| API
    FE -->|Login / Token| Cognito
    API -->|Validate JWT| Cognito
    API --> BL
    BL --> DB
    BL -.->|Generate PDF| S3
    API --> Logs
    BL --> Logs

    style TrustBoundary fill:#f9f9f9,stroke:#333
    style AuthBoundary fill:#e8f4fd,stroke:#0366d6
    style AppBoundary fill:#e6ffed,stroke:#22863a
Component explanation:
	•	React SPA: only UI and client-side validation; never trusted for security decisions.
	•	Cognito: sole source of identity; issues short-lived JWTs.
	•	FastAPI: single entry point, request validation, auth middleware, orchestration.
	•	Business Logic module: pure Python functions for score calculation and action validation (framework-agnostic).
	•	PostgreSQL: source of truth for users, actions, scores, audit trail.
	•	S3: optional for generated reports (can be deferred and served as JSON/CSV for demo).
	•	CloudWatch: structured logs + basic alarms; no PII in log messages.
Primary data flow is left-to-right. Failure paths (auth failure, DB down, invalid action) return clean 4xx/5xx with no stack traces and are logged server-side only.

Codebase Folder Tree
credit-gym/
├── frontend/                  # React + Vite
│   ├── src/
│   │   ├── components/
│   │   ├── pages/             # Login, Dashboard, LogAction, Report
│   │   ├── hooks/
│   │   ├── api/               # typed API client
│   │   └── types/
│   ├── public/
│   └── package.json
├── backend/
│   ├── app/
│   │   ├── main.py            # FastAPI app + middleware
│   │   ├── api/               # route modules (auth, actions, scores, reports)
│   │   ├── core/              # config, security, dependencies
│   │   ├── domain/            # pure business logic (score engine, validators)
│   │   ├── models/            # SQLAlchemy / Pydantic models
│   │   ├── repositories/      # data access only
│   │   ├── services/          # orchestration (action service, report service)
│   │   └── schemas/           # request/response Pydantic models
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   ├── alembic/               # migrations
│   ├── requirements.txt
│   └── Dockerfile
├── .github/workflows/         # simple CI
├── docker-compose.yml         # local Postgres + backend
├── .env.example
└── README.md
Dependency direction (strict): api → services → domain + repositories → models domain has zero knowledge of FastAPI, SQLAlchemy, or AWS. Repositories are the only place that talks to the database. External integrations (Cognito JWT validation, future OCR) live behind thin service adapters.
Modules that can be developed & tested independently: domain (pure unit tests), repositories (integration with test DB), services (mocked repos).

Core API Contracts and Data Model
Minimal data model (Postgres):
	•	users: id (uuid), cognito_sub, email, display_name, age_band, is_student, created_at, consent_version
	•	actions: id, user_id, action_type (enum: payment, savings, education, other), amount (nullable), evidence_ref (nullable), occurred_at, points_awarded, created_at
	•	scores: user_id (PK), current_score, last_calculated_at, version
	•	audit_log: id, user_id, event_type, payload (jsonb), created_at
Core endpoints (all require valid JWT except health & login redirect):
POST   /api/v1/actions          # body: {action_type, amount?, evidence_ref?, occurred_at}
GET    /api/v1/actions          # list own actions (paginated)
GET    /api/v1/score            # current score + breakdown
POST   /api/v1/reports          # generate report → returns download URL or JSON
GET    /api/v1/health
Pydantic models enforce enums, date ranges, maximum amounts, and string lengths at the boundary.
Score rules (example, adjustable):
	•	On-time payment simulation: +15
	•	Savings action ≥ R100: +10
	•	Completed education module: +20
	•	Cap daily points and total score to prevent gaming.

Security and Failure-Handling Matrix
Risk / Failure Scenario
Mitigation
Implementation Location
Verification
Unauthenticated request
JWT required on every protected route; 401
FastAPI dependency + Cognito
Unit + e2e auth tests
Authorisation bypass
Resource ownership checks (user_id == JWT sub)
Service layer
Integration tests with two users
Invalid / malicious input
Pydantic strict models + length/type checks
API schemas
Property-based + unit tests
SQL injection
SQLAlchemy ORM / parameterised queries only
Repositories
Static analysis + manual review
Secrets in code / logs
Env vars + AWS Secrets Manager; never log tokens
core/config.py + logging filter
CI secret scan + manual
XSS / CSRF
React escapes by default; SameSite cookies if used; CORS whitelist
Frontend + FastAPI CORS
Manual + OWASP ZAP light scan
Rate / abuse
Basic rate limit middleware (e.g. slowapi)
API middleware
Load test
DB unavailable
Connection pool timeout → 503; no partial writes
Repository + FastAPI exception handler
Chaos test (kill DB)
External service timeout (Cognito)
Explicit timeouts; fail closed on auth
Auth dependency
Mock timeout tests
Duplicate action / race
Unique constraint + idempotency key on actions
DB + service
Concurrent test
POPIA / data minimisation
Collect only needed fields; consent recorded; retention policy documented
Domain + models
Privacy review checklist
Report generation fails
Fallback to JSON export; never claim success
Report service
Failure-path test
Remaining risks that must be independently verified before real users: formal threat model, penetration test, DPIA for POPIA, age-verification process for minors, and real evidence attestation.

Testing Plan
	•	Unit: domain score engine (all edge cases: caps, negative amounts, future dates), validators, pure functions.
	•	Integration: repository CRUD against test Postgres; JWT validation with Cognito test user pool or mocked JWKS.
	•	API contract: httpx tests for every endpoint (status codes, schemas, ownership).
	•	Security: unauthenticated access, cross-user access, oversized payloads, invalid enums.
	•	Failure: DB connection refused, Cognito timeout, malformed JWT, concurrent duplicate actions.
	•	E2E (happy path): Playwright or simple script that logs in → logs action → checks score → generates report.
	•	Acceptance criterion for any feature: unit + relevant integration tests green, no secrets in logs, ownership checks pass, and the primary user journey can be demonstrated live.
Never run destructive tests against the demo/production database.

Deployment Plan
Demo (within remaining hours):
	1	docker-compose up for local Postgres + backend.
	2	Frontend npm run build → serve from S3 + CloudFront or even Vite preview.
	3	Cognito user pool (free tier) + App Client.
	4	Environment variables via .env / App Runner secrets.
	5	Single command deploy script or GitHub Actions that builds the Docker image and pushes to App Runner / ECR.
Minimal CI:
	•	Lint (ruff + eslint)
	•	Type check
	•	Unit + integration tests
	•	Build Docker image
Production gaps to close later:
	•	Multi-AZ RDS + automated backups
	•	WAF + stricter rate limits
	•	Secrets Manager rotation
	•	Proper observability dashboards + alerts
	•	Formal backup/restore tested
	•	Legal review of terms + privacy notice
	•	Staging environment that mirrors production
Rollback: App Runner / Elastic Beanstalk supports previous versions; database migrations are forward-only with careful review.

Prioritised Implementation Checklist
	1	Scaffold + health Empty FastAPI + React + Docker Compose + Cognito skeleton. Acceptance: /health returns 200; frontend loads.
	2	Auth end-to-end Login → JWT → protected route that returns current user. Tests: unauthenticated 401, valid token succeeds.
	3	Data model + migrations Users, actions, scores tables via Alembic. Acceptance: can insert a user and query it.
	4	Log action (vertical slice) Authenticated POST /actions → persists → returns points awarded. Domain score calculation unit-tested. Acceptance: action appears in DB and score updates.
	5	View score + history GET endpoints; frontend dashboard shows score and last 5 actions.
	6	Report generation Simple JSON or PDF (reportlab or weasyprint) of the user’s history + score. Can be deferred to pure JSON if time is short.
	7	Hardening pass Rate limiting, structured logging, CORS lockdown, input size limits, ownership checks, basic error handling.
	8	Seed data + demo script Pre-populate 2–3 example users so judges can see progress without waiting.
	9	Deploy to AWS Frontend static + backend container + RDS. Final smoke test of the full journey.
Stop and review before any schema change that drops columns or any change that weakens auth.

Assumptions and Unresolved Risks
Assumptions:
	•	Real credit-bureau or bank-feed integration is out of scope for the 24 h window.
	•	Self-reported actions + timestamps are sufficient for a credible demo narrative.
	•	Users are primarily South African students; age is self-declared.
	•	One developer is comfortable with Python + basic React + AWS console.
Unresolved risks that must be verified:
	•	Exact Cognito configuration and JWT validation edge cases under time pressure.
	•	Whether af-south-1 has the required free-tier capacity on the day.
	•	Legal wording for consent and “this is not a formal credit score”.
	•	How under-18 users will be handled (parental consent flow is currently a flag only).
	•	Long-term score gaming and evidence authenticity once real users arrive.
	•	Cost of RDS + data transfer if the demo is left running for weeks.
This architecture delivers the core value (build and show a credibility record) with the least moving parts, keeps security and testability first-class, and gives a clear, low-risk path from empty repository to a live demo and onward to production.
