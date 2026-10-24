Hackathon Engineering Blueprint: Credit Gym for Young People

Architecture proposal · Python · AWS · South Africa · 24-hour hackathon · One developer

1. Architecture summary

The core product decision

Build a Credit Gym that helps young people establish a verifiable record of responsible financial behaviour before they qualify for conventional credit.

The MVP should let a young person complete a small set of financial responsibility activities, record their progress, and view a transparent Credit Readiness Profile showing their demonstrated habits.

There is one important distinction: completing activities in your app does not automatically create a recognised credit score or establish a credit history with a South African credit bureau. The prototype must communicate that distinction honestly.

THE MVP’S SINGLE MOST IMPORTANT FUNCTION

Turn responsible financial behaviour into a transparent, verifiable readiness record.

* A student registers and signs in.
* They complete a simple financial responsibility activity, such as a budgeting exercise or a simulated repayment commitment.
* The backend validates the activity and records the outcome.
* The student sees their updated readiness profile, activity history, and the evidence behind their progress.

What to build in 24 hours

Build one complete vertical slice, not a miniature banking platform.

Essential MVP

Authentication, one activity, validated evidence, persisted progress, and a readiness dashboard.

Safety boundary

No real lending, credit decisions, bank connections, or claims of a recognised credit score.

Persistent evidence

Store activity results in a database. Never invent a successful result if persistence fails.

Demonstrable outcome

A judge can complete an activity, refresh the page, and see the saved record.

Recommended architecture

Use a modular monolith: one Python backend, one relational database, and a lightweight web frontend. Deploy the backend and database using managed AWS services where practical.

* Frontend: React with TypeScript, if the developer is comfortable with it; otherwise use server-rendered HTML and templates to minimise implementation time.
* Backend: Python with FastAPI and Pydantic.
* Database: PostgreSQL, accessed through SQLAlchemy and managed migrations.
* Authentication: Amazon Cognito, or a well-tested existing authentication integration.
* Hosting: AWS App Runner for the Python container, or an already configured equivalent; Amazon RDS for production-grade relational persistence.
* Testing: pytest, FastAPI’s test client, and integration tests against a disposable test database.

Do not introduce microservices, AI agents, a credit bureau integration, or a message queue for this demonstration. None is necessary to validate the core proposition.

2. Technology stack

The following is one coherent stack, not a menu of technologies to experiment with. The goal is to spend the remaining time building the product rather than configuring infrastructure.

Layer	Recommendation	Rationale and trade-offs
Frontend	React + TypeScript + Vite	Good for a polished interactive dashboard. If React is unfamiliar, use FastAPI templates instead.
Backend/API	Python + FastAPI + Pydantic	Type-friendly request validation, API documentation, and straightforward automated testing.
Database	PostgreSQL + SQLAlchemy	Relational integrity for users, activities, and evidence.
Migrations	Alembic	Version-controlled schema changes and repeatable deployment.
Authentication	Amazon Cognito	Managed identity, password handling, and token issuance. More setup than a mocked demo, but better for a real-user path.
Hosting	AWS App Runner	Container-based Python hosting with managed scaling. Alternatively, use an AWS service already configured by the team.
Database hosting	Amazon RDS for PostgreSQL	Managed database, backups, and operational controls. For the hackathon, a disposable local PostgreSQL database may be faster.
Secrets	AWS Secrets Manager or deployment-managed secrets	Keep credentials out of source control and frontend bundles.
Testing	pytest, HTTPX, Testcontainers where practical	Business-rule tests, API contract tests, and reproducible database integration tests.
Code quality	Ruff, mypy or Pyright, pip-audit	Linting, type checks, and dependency vulnerability checks.
CI/CD	GitHub Actions	Run checks before deployment; avoid elaborate pipelines during the first few hours.
Monitoring	Python structured logging, CloudWatch, health endpoint	Detect errors and operational failures without building a separate observability platform.

Cost and deployment trade-offs

* Fastest demo: Run the frontend and API locally, use PostgreSQL, and deploy only when the primary journey works. Use synthetic data.
* AWS demo: Deploy a container to App Runner and connect to a suitable PostgreSQL instance. Do not assume the database will be free or that all AWS services share a free tier.
* Production: Add private database networking, restricted IAM permissions, managed secrets, automated backups, restore testing, alerting, and a controlled release process.

AWS App Runner charges for provisioned memory and active compute, and additional AWS services incur their own costs. The published examples on its pricing page use particular regions and configurations, so calculate the estimate for the actual deployment region rather than treating an example price as a quote.

Important: AWS is not a single security boundary. The application, database, identity service, logs, deployment credentials, and network configuration each require their own access controls.

3. Mermaid architecture diagram

The diagram shows the proposed production-oriented design. For the hackathon, implement only the components needed to complete the core journey.

flowchart TD
    U["Student"]
    subgraph PUBLIC["Public trust boundary"]
        FE["Web frontend<br/>React + TypeScript"]
    end
    subgraph APP["Application trust boundary"]
        AUTH["Amazon Cognito<br/>Identity and tokens"]
        API["FastAPI<br/>Authentication and request validation"]
        SVC["Credit Gym service<br/>Activity rules and readiness calculation"]
        REPO["Repository layer<br/>SQLAlchemy"]
    end
    subgraph DATA["Restricted data boundary"]
        DB[("PostgreSQL<br/>Users, activities, evidence")]
    end
    subgraph OPS["Operations"]
        LOG["Structured logs and<br/>CloudWatch monitoring"]
        SEC["Secrets Manager<br/>Deployment configuration"]
    end
    U --> FE
    FE -->|"Sign in"| AUTH
    FE -->|"HTTPS + access token"| API
    API -->|"Verify identity and ownership"| AUTH
    API --> SVC
    SVC -->|"Validated transaction"| REPO
    REPO --> DB
    API -.->|"Sanitised errors and metrics"| LOG
    SVC -.->|"Operational events"| LOG
    SEC -.->|"Runtime credentials"| API
    API -.->|"401 / 403: deny access"| FE
    DB -.->|"Unavailable / transaction fails"| API
    API -.->|"Generic error; no false success"| FE

Component responsibilities

* Frontend: Collects input and renders the student’s readiness profile. It is not trusted to calculate official scores or enforce access permissions.
* Cognito: Authenticates users. The backend validates tokens and enforces authorisation for each protected operation.
* FastAPI: Provides API endpoints, validates request schemas, enforces limits, and translates exceptions into safe HTTP responses.
* Credit Gym service: Implements the actual business rules: whether an activity is valid, whether it has already been completed, and how the profile changes.
* Repository: Contains database access operations. It should not decide business eligibility or independently invent readiness results.
* PostgreSQL: Stores durable records and enforces database constraints.
* Operations and secrets: Logs operational information without exposing personal financial details. Runtime credentials are not embedded in the application code.

Failure behaviour: If the database transaction fails, the API must not tell the user that the activity was recorded. If identity or ownership cannot be verified, the operation must be denied.

The MVP does not need an external AI provider. A deterministic, explainable readiness calculation is easier to test and defend to judges.

4. Modular codebase structure

Use a structure that separates business rules from HTTP handling and database access, without building an elaborate framework inside the framework.

credit-gym/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── api/
│   │   └── types/
│   ├── package.json
│   └── vite.config.ts
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── api/
│   │   │   ├── dependencies.py
│   │   │   ├── errors.py
│   │   │   └── routes/
│   │   │       ├── health.py
│   │   │       ├── activities.py
│   │   │       └── profile.py
│   │   ├── core/
│   │   │   ├── security.py
│   │   │   └── logging.py
│   │   ├── domain/
│   │   │   ├── models.py
│   │   │   └── rules.py
│   │   ├── schemas/
│   │   │   ├── activity.py
│   │   │   └── profile.py
│   │   ├── services/
│   │   │   ├── activity_service.py
│   │   │   └── readiness_service.py
│   │   ├── repositories/
│   │   │   ├── activity_repository.py
│   │   │   └── profile_repository.py
│   │   └── integrations/
│   │       └── cognito.py
│   ├── migrations/
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   ├── api/
│   │   └── security/
│   ├── alembic.ini
│   ├── pyproject.toml
│   ├── Dockerfile
│   └── .env.example
│
├── .github/
│   └── workflows/
│       └── ci.yml
├── docker-compose.yml
├── .gitignore
├── README.md
└── SECURITY.md

Dependency direction

flowchart TD
    ROUTE["API routes"]
    SERVICE["Application services"]
    DOMAIN["Domain rules"]
    REPO["Repository interfaces"]
    DB["PostgreSQL adapter"]
    AUTH["Cognito adapter"]
    ROUTE --> SERVICE
    SERVICE --> DOMAIN
    SERVICE --> REPO
    REPO --> DB
    ROUTE --> AUTH

* Domain rules must not import FastAPI, SQLAlchemy, or AWS SDKs.
* Services orchestrate business operations and transactions.
* Routes handle HTTP concerns and delegate to services.
* Repositories encapsulate persistence.
* Integrations isolate provider-specific code so it can be replaced or mocked.

For a 24-hour build, these are logical boundaries, not a mandate to create dozens of files before the first working endpoint. Start with the essential modules and extract a separate module when it improves clarity or testability.

5. Core API contracts and data model

The smallest useful data model consists of three entities: a user profile, an activity definition, and an activity record.

Data model

erDiagram
    USER_PROFILE ||--o{ ACTIVITY_RECORD : owns
    ACTIVITY_DEFINITION ||--o{ ACTIVITY_RECORD : describes
    USER_PROFILE {
        uuid id PK
        string cognito_subject UK
        datetime created_at
    }
    ACTIVITY_DEFINITION {
        string code PK
        string title
        string description
        boolean active
    }
    ACTIVITY_RECORD {
        uuid id PK
        uuid user_id FK
        string activity_code FK
        string status
        json evidence
        datetime completed_at
        string idempotency_key
    }

Design decisions:

* Use Cognito’s immutable subject identifier to associate the authenticated identity with a local profile. Never trust a user ID supplied by the browser as proof of ownership.
* Keep activity definitions separate from individual user records.
* Store only evidence required to demonstrate the activity. Do not collect identity numbers, bank credentials, or transaction histories for the prototype.
* Make activity creation transactional and prevent accidental duplicate completion.
* Use a unique database constraint on (user_id, idempotency_key) if idempotency keys are implemented.
* Treat the evidence JSON as structured, validated data—not an arbitrary place to store sensitive information.

API contracts

Method and endpoint	Purpose	Success response	Key failure cases
GET /health	Basic application health	200 {“status”:“ok”}	503 if a required dependency is unhealthy
GET /v1/activities	List active activities	200 {“items”:[…]}	401 if authentication is required but missing
POST /v1/activities/{code}/complete	Record an activity	201 with activity record	401, 404, 409, 422, 503
GET /v1/profile	Retrieve own readiness profile	200 with progress and evidence summary	401, 503

Require authentication for all personal data endpoints. Public health checks should expose no infrastructure details; if deeper dependency health is needed, restrict that endpoint.

Example completion request:

{
  "idempotency_key": "client-generated-unique-value",
  "evidence": {
    "monthly_budget_completed": true
  }
}

Example success response:

{
  "activity_id": "generated-uuid",
  "activity_code": "budget_basics",
  "status": "completed",
  "readiness_progress": {
    "completed_activities": 1,
    "total_activities": 3
  }
}

The values above illustrate the contract; they are not evidence of an actual implementation. In the real API, the server validates the activity code, validates evidence against the activity’s schema, checks the authenticated user’s ownership, and computes progress from persisted records.

A repeated request using the same idempotency key should return the existing result when its meaning is identical. It must not create another record. Reusing the key for a conflicting operation should return a conflict response.

6. Security and failure-handling matrix

Risk or failure	Mitigation and implementation location	Verification
Missing or invalid identity token	Validate token signature, issuer, audience, expiry, and applicable claims in authentication dependencies.	Test missing, expired, malformed, and wrong-audience tokens.
Access to another student’s records	Derive the owner from the verified identity on the server. Apply ownership checks in service and repository queries.	Attempt cross-user profile and activity access.
Invalid or malicious input	Pydantic schemas, length limits, allowed-value checks, and database constraints.	Test malformed JSON, oversized values, unexpected fields, and injection payloads.
SQL injection	SQLAlchemy parameterised operations; never construct SQL from untrusted strings.	Test malicious input and review database queries.
XSS and CSRF	Escape rendered content; use secure cookie settings and CSRF protection if cookie-based authentication is used. Configure CORS narrowly.	Test untrusted text rendering and cross-origin requests.
Credential leakage	Use environment configuration locally and managed secrets in AWS. Keep secrets out of logs, frontend code, Git, and error responses.	Secret scanning and deployment configuration review.
Database outage	Explicit connection and query timeouts, transaction rollback, safe 503 responses.	Disable the test database and verify no false success.
Duplicate submission	Idempotency key, unique constraint, and transaction-safe service logic.	Submit identical requests concurrently.
External service timeout	Explicit timeouts; bounded retries only for safe operations. No unnecessary external services in the MVP.	Simulate timeout and malformed responses.
Missing configuration or expired credentials	Fail startup or readiness checks when essential configuration is absent; never substitute insecure defaults.	Remove required configuration and confirm controlled failure.
Unexpected exception	Centralised exception handling, correlation IDs, sanitised logs, and generic client errors.	Force an exception and check the response and logs.
Abuse or automated submissions	Rate limits, request size limits, sensible activity frequency constraints, and monitoring.	Load and rate-limit tests.
Dependency vulnerability	Lock dependencies, review updates, and run package vulnerability scanning.	CI dependency audit and remediation review.
Personal data exposure	Data minimisation, restricted database access, encryption in transit, appropriate encryption at rest, retention rules, and controlled access to logs.	Privacy review, access-control tests, and configuration audit.

Failure policy

Scenario	User experience	Data integrity and retry
Invalid input	Clear field-level validation message	No write; user may correct and resubmit.
Unauthenticated or unauthorised request	Sign-in prompt or access-denied message	No protected data returned and no write performed.
Database unavailable	“We couldn’t save your progress. Please try again.”	Roll back the transaction; retry safely using the same idempotency key.
Duplicate request	Show the existing completion result	Return the existing record; do not duplicate the activity.
Unexpected error	Generic message with a reference ID	Log diagnostic context without sensitive payloads.
External provider failure	Explain that the operation could not be verified	Never create a successful evidence record based on a timeout or fabricated response.

South African privacy and credit regulation

This product sits close to a regulated financial-services boundary. Resolve these questions before moving beyond a synthetic-data prototype.

* POPIA: Determine the lawful basis for processing, provide a clear privacy notice, minimise collection, define retention and deletion procedures, restrict access, and assess operator agreements and cross-border processing. The Information Regulator’s security-safeguard guidance emphasises appropriate measures, risk management, and verification of their effectiveness.
* National Credit Act (NCA): Obtain South African legal advice on whether the intended service, data collection, reports, or commercial model triggers credit-provider or credit-bureau registration requirements. The NCA regulates consumer credit and credit information, and section 43 addresses credit-bureau registration.
* Recognised credit reporting: Do not promise that a simulated repayment or app-generated readiness score will be accepted by lenders or reported to credit bureaus. A future evidence-sharing model needs an appropriate partner, lawful data flows, verification, and clear consumer rights.
* Minors: Define an appropriate minimum age and obtain specialist advice on processing children’s personal information, parental consent requirements where applicable, and safeguards for vulnerable users.
* Automated decisions: Do not use an unvalidated algorithm to label young people as creditworthy or unworthy. Keep the initial readiness assessment transparent and educational.

Relevant official references: South African National Credit Act⁠￼ and Protection of Personal Information Act⁠￼.

These controls reduce risk; they do not establish legal compliance or prove the application secure. Independent security testing and legal review are still required before handling real financial records or making credit-related claims.

7. Testing plan

Tests should be built alongside each vertical slice rather than left until the end.

Test layer	Essential cases	Acceptance criterion
Unit	Activity eligibility, readiness calculation, duplicate rules, invalid evidence	Deterministic outputs for valid and invalid cases
API	Authentication, schema validation, status codes, response shapes	API matches the documented contracts
Integration	PostgreSQL persistence, constraints, migrations, rollback	Records survive restart and failed transactions do not leave partial state
End-to-end	Sign in, complete activity, view updated profile, refresh	The same persisted result appears after refresh
Security	Cross-user access, invalid tokens, injection, XSS, rate limits	Unauthorised operations fail without data leakage
Failure-path	Database outage, expired credentials, unexpected exceptions	Safe errors, no fabricated success, no sensitive diagnostics in client responses
Delivery	Build, lint, type checks, migrations	A clean environment can reproduce the application and its checks

Feature completion gates

Feature 1 — Authentication

Failure case: an expired token attempts to retrieve a profile. Test: API rejects the request and returns no profile data. Complete when valid identity is verified server-side.

Feature 2 — Activity completion

Failure case: malformed evidence or a duplicate submission. Test: invalid evidence creates no record; duplicate requests create at most one. Complete when a valid activity is durably recorded.

Feature 3 — Readiness profile

Failure case: the database is unavailable. Test: the endpoint returns a controlled error rather than invented progress. Complete when the profile reflects only persisted, validated records.

Use a disposable PostgreSQL test database for integration tests and mocked adapters for unit tests. Never run destructive tests against production data. For the demo, report only test results actually observed.

8. Deployment and delivery strategy

Hackathon deployment path

Use the shortest path that produces a repeatable, demonstrable result.

* Local development
    Run FastAPI, the frontend, and PostgreSQL locally. Keep a .env.example in Git, exclude real .env files, and use synthetic student accounts.
* Automated verification
    Configure GitHub Actions to run linting, type checks, unit tests, and API tests on each pull request. Add integration tests against a disposable database.
* Demo deployment
    Package the API in Docker and deploy it to AWS App Runner if the account and permissions are ready. Host the frontend separately or serve the compiled frontend through the backend to reduce configuration overhead.
* Persistent storage
    Provision PostgreSQL, apply Alembic migrations, restrict network access, and verify that records survive an application restart.
* Operational verification
    Check HTTPS, authentication, activity completion, database persistence, health checks, logs, and the actual deployed URL. Do not describe deployment as successful until the live journey has been tested.

Development versus production

Concern	Hackathon demo	Before production
Data	Synthetic users and activities	Verified privacy basis and data governance
Authentication	Cognito if integration is feasible; otherwise a clearly labelled local-only demo mode	Managed identity with reviewed token validation and recovery flows
Database	Local PostgreSQL or a disposable managed instance	Restricted production database, backups, restore tests, migration controls
Secrets	Local environment configuration	Managed secrets, IAM least privilege, rotation
Network	Local or HTTPS-hosted demo	Restricted ingress and egress, private database connectivity
CI/CD	Basic automated tests and manual release	Controlled deployments, approval gates, rollback procedure
Monitoring	Health endpoint and basic logs	Alerts, audit events, error monitoring, incident response
Resilience	Explicit timeouts and transaction rollback	Tested recovery, capacity limits, backup restoration, availability objectives

Cost control: Set AWS billing alerts and a budget before provisioning resources. Stop or delete disposable services after the event, and verify whether database backups, storage, networking, and log retention continue to incur charges.

Do not add Kubernetes, multi-region deployment, a service mesh, or elaborate infrastructure-as-code merely to make the architecture look sophisticated. Introduce them only when a demonstrated requirement justifies the cost.

9. Prioritised implementation checklist

With 24 hours and one Python developer, time-box infrastructure and prioritise the end-to-end user journey.

24-hour build plan

Indicative allocation. If time runs short, remove optional scope instead of weakening security or data integrity.

1. Define the contract and scaffold

1.5 h

Create the repository, FastAPI app, configuration, README, activity definition, and API contracts.

2. Implement business rules

2 h

Implement evidence validation, activity eligibility, progress calculation, and unit tests.

3. Implement persistence

3 h

Create SQLAlchemy models, migrations, repositories, constraints, and database integration tests.

4. Complete the API vertical slice

3 h

Implement activity listing, completion, profile retrieval, transaction handling, and error responses.

5. Add identity and authorisation

2.5 h

Integrate Cognito if feasible; otherwise keep the prototype local-only and clearly label its limitations. Never create a publicly accessible fake authentication mechanism.

6. Build the user experience

4 h

Create the activity screen, completion feedback, readiness dashboard, loading states, and safe error messages.

7. Test and harden

3 h

Test the full journey, duplicate requests, access control, database failure, and configuration errors.

8. Deploy and rehearse

3 h

Deploy if feasible, test the live URL, prepare synthetic demo data, and rehearse a clean walkthrough.

Total

22 h

The remaining 2 hours are a contingency buffer. If Cognito or AWS setup becomes a time sink, retain the tested local journey and cut optional features.

Incremental coding protocol for the AI assistant

Use this sequence for every implementation task.

* Explain the proposed change, affected files, assumptions, security implications, and tests.
* Make the smallest coherent change necessary.
* Review the diff for duplicated logic, insecure defaults, unnecessary dependencies, and unrelated edits.
* Run the relevant tests and report the actual command and result.
* Integrate the change with the existing application and verify its acceptance criterion.
* Stop for human review before destructive migrations, security-control changes, production-secret handling, or irreversible data operations.

Do not ask an AI coding assistant to generate the entire repository in one pass. Generate one vertical slice, inspect it, test it, and then proceed.

Prioritised task board

Implementation tracker

0 of 9

Define one activity and its evidence rules

Specify valid inputs, completion conditions, and readiness contribution.

Scaffold FastAPI and automated tests

Create configuration, health endpoint, test setup, and API schemas.

Implement domain rules and unit tests

Keep calculations deterministic and independent of infrastructure.

Persist activities in PostgreSQL

Add models, migrations, transaction handling, and duplicate protection.

Implement the three core API operations

List activities, complete an activity, and retrieve the authenticated user’s profile.

Implement authentication and ownership checks

Validate identity server-side and prevent cross-user access.

Build the complete frontend journey

Show activity, success state, saved evidence, and updated progress.

Test failure paths and security boundaries

Cover invalid input, duplicate requests, database outage, and unauthorised access.

Deploy and rehearse

Verify the live URL, persistence, configuration, logs, and demo walkthrough.

10. Assumptions and unresolved risks

Assumptions for the initial build

* The target users are South African students or young adults with little or no established credit history.
* The product initially provides education and a record of simulated or independently verifiable activities.
* No money is lent, collected, transferred, or held by the MVP.
* No bank account, credit bureau, or third-party financial data integration is required for the demonstration.
* The readiness profile is an educational indicator, not a credit score, lending recommendation, or guarantee of approval.
* A single developer is responsible for implementation, testing, and the demonstration.

Questions to resolve before expanding the product

Question	Why it matters
What evidence makes an activity credible?	A completed quiz proves knowledge, not repayment behaviour. The product must distinguish self-reported, simulated, and independently verified evidence.
Who will recognise the record?	Lenders need to trust the source, verification method, relevance, and integrity of the evidence.
How will the product ultimately generate recognised credit history?	A partnership or regulated reporting arrangement may be needed; an internal score alone does not solve this.
Will the product eventually facilitate lending?	Loans, repayments, fees, and credit agreements introduce additional regulatory, operational, and consumer-protection obligations.
How will young people’s information be protected?	Consent, lawful processing, access rights, retention, deletion, security incidents, and children’s information require careful assessment.
How will the product avoid misleading users?	Explain what a readiness profile measures, what it cannot prove, and whether any third party recognises it.

The biggest product risk

The engineering challenge is manageable. The harder problem is creating evidence that a future lender considers trustworthy and useful.

A polished dashboard can demonstrate the experience, but it cannot by itself establish that the student’s behaviour predicts repayment or that a lender will recognise the record.

For the hackathon, demonstrate the mechanism honestly. For the business, validate the evidence model with potential lenders, a recognised credit bureau, and South African regulatory advisers before promising that the product builds recognised credit histories.

Definition of done

The MVP is ready to demonstrate when a test student can authenticate, complete one activity, see the backend validate and persist the record, and retrieve the same result after refreshing the application. Unauthorised access must be rejected, duplicate submissions must not create duplicate records, and database failure must never be presented as successful completion.

The next development step is to implement that single journey—not to build every future capability at once.