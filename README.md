# I-ITSM — Intelligent IT Service Management & Incident Resolution System

A full-stack, enterprise-grade IT Service Management (ITSM) and Incident Resolution platform that manages the complete lifecycle of IT incidents:
$$\text{Report} \longrightarrow \text{Classify} \longrightarrow \text{Prioritise} \longrightarrow \text{Assign} \longrightarrow \text{Investigate} \longrightarrow \text{Resolve} \longrightarrow \text{Verify} \longrightarrow \text{Close}$$

Engineered for **CS F213 / MAC F212 Object-Oriented Programming (BITS Pilani, Sem I 2026–27)**. The architecture strictly models object-oriented principles, design patterns, and enterprise software engineering standards. Source files are annotated with semantic OOP tags (such as `// [OOP: Polymorphism]` or `// [Pattern: State]`) for traceable architectural verification.

| Technology | Stack Specification |
|---|---|
| **Backend** | Java 21 · Spring Boot 3.5 · Spring Security (JWT) · Spring Data JPA / Hibernate · Flyway · WebSocket (STOMP) · Virtual Threads |
| **Frontend** | React 19 · TypeScript 5.9 · Vite · TanStack Query · Tailwind CSS 4 · Recharts · Lucide Icons |
| **Database** | PostgreSQL 17 / 18 (Neon Serverless Cloud · H2 compatibility mode for offline test runner) |
| **DevOps & Cloud** | Docker Compose · Vercel (SPA) · Render (Docker Web Service) · GitHub Actions CI · OpenAPI / Swagger UI · Actuator |
| **Verification** | 71 Backend tests (JUnit 5 + Mockito + MockMvc, ~79% instruction coverage) · 9 Frontend tests (Vitest) |

---

## Live Cloud Deployment

The application is deployed and publicly accessible in the cloud without requiring local installation:

- **Web Application**: Available on Vercel
- **Interactive API Documentation (Swagger UI)**: [`https://itsm-backend-q72n.onrender.com/swagger-ui.html`](https://itsm-backend-q72n.onrender.com/swagger-ui.html)
- **OpenAPI 3.1 Specification**: [`https://itsm-backend-q72n.onrender.com/v3/api-docs`](https://itsm-backend-q72n.onrender.com/v3/api-docs)

> [!NOTE]
> **Cloud Free-Tier Notice**: The backend is hosted on Render's free tier and Neon serverless PostgreSQL. If the service has been idle, the initial login request may take ~30–45 seconds to spin up the container. All subsequent interactions are instant.

---

## Pre-Seeded Test Accounts

The platform comes pre-seeded with hierarchical user roles. The sign-in page includes one-click autofill credentials for each role:

| Role | Email | Password | Scope & Responsibilities |
|---|---|---|---|
| **Employee** | `employee@itsm.local` | `Password@123` | End-user self-service: report incidents, track status, provide feedback |
| **IT Technician** | `tech@itsm.local` | `Password@123` | Operational queue (`My queue`), acknowledge, investigate, request info, resolve |
| **Team Lead** | `lead@itsm.local` | `Password@123` | Team oversight, reassignment, workload balancing, Level 1 escalation |
| **Administrator** | `admin@itsm.local` | `Password@123` | Full system administration, SLA policies, catalog, audit logs, strategy config |
| **IT Manager** | `manager@itsm.local` | `Password@123` | Executive dashboard, breach analytics, major incident coordination, Level 2 escalation |

*Additional seeded technicians (`sneha.patel`, `rahul.verma`, `kavya.nair`, `rohan.das`, `meera.joshi`) and employees are pre-loaded under the `@itsm.local` domain.*

---

## Local Execution & Setup

### Option A — Docker Compose (Full Stack)

To run the complete production-like stack locally (PostgreSQL, Mailpit SMTP catcher, Spring Boot backend, and React frontend):

```bash
docker compose up --build
```

Access points:
- **Web Application**: `http://localhost:3000`
- **Swagger UI**: `http://localhost:8080/swagger-ui.html`
- **Mailpit Web UI (Local SMTP Catcher)**: `http://localhost:8025`

### Option B — Local Development (Java 21 + Node 20+)

Run the backend and frontend in separate terminals:

```bash
# Terminal 1 — Backend on :8080 (seeded in-memory H2 database)
cd backend && ./mvnw spring-boot:test-run

# Terminal 2 — Frontend on :5173 (Vite dev server with API proxy)
cd frontend && npm install && npm run dev
```

---

## Incident Resolution Lifecycle & Core Workflows

The platform automates the end-to-end incident management lifecycle through three integrated workflows:

```
[Reported] ──> [Classified] ──> [Prioritised] ──> [Assigned]
                                                      │
    ┌─────────────────────────────────────────────────┘
    ▼
[Investigating] <──> [Pending Info (SLA Paused)]
    │
    ▼
[Resolved] ──> [Feedback / Verification] ──> [Closed]
```

### 1. Incident Intake, Automated Prioritisation & Intelligent Assignment
1. An employee submits an incident specifying title, description, affected service, impact, and urgency.
2. The system executes live pre-submission analysis:
   - **Automated Classification**: Derives category via keyword scoring across title, description, and service defaults.
   - **Priority Assessment**: Evaluates priority via Matrix calculation ($\text{Impact} \times \text{Urgency} \times \text{Criticality}$) or Weighted Strategy.
   - **SLA Binding**: Associates targeted response and resolution deadlines, incorporating per-service overrides.
3. The configured assignment strategy (Skill-based, Load-balanced, Round-robin, or Smart Scoring) automatically assigns the optimal technician.
4. Real-time notifications are dispatched via WebSocket (in-app toast), email, and SMS (for critical P1 incidents).

### 2. Investigation, SLA Tracking & Resolution Verification
1. The assigned technician acknowledges the incident, recording the initial response SLA timestamp.
2. If further details are required from the reporter, the technician moves the incident to `PENDING_INFO`. The SLA countdown **automatically pauses**, preventing unfair SLA breaches during user delays.
3. Upon user response, the SLA clock resumes with an adjusted deadline.
4. Intelligent root-cause analysis provides historical resolution suggestions.
5. The technician marks the ticket `RESOLVED`. The reporter verifies the resolution, submits star ratings and satisfaction comments, and the incident transitions to `CLOSED`.

### 3. Incident Correlation, Duplicate Detection & Hierarchical Escalation
1. Incoming tickets are analyzed against active incidents using TF-IDF text similarity and domain synonym mapping.
2. Matches above threshold are flagged as duplicate candidates and can be merged into a parent incident.
3. Incidents clustering around related services within a sliding time window trigger a **Major Incident Cluster** alert to administrators.
4. Breached or unacknowledged tickets trigger automated escalation through a Chain of Responsibility:
   $$\text{Assigned Technician} \longrightarrow \text{Team Lead} \longrightarrow \text{Administrator} \longrightarrow \text{IT Manager}$$

---

## Compliance with Course Specifications

| Requirement Area | Specification Requirement | Project Implementation |
|---|---|---|
| **User Hierarchy** | Employee, Technician, Administrator | 5 distinct roles (`User` $\rightarrow$ `Employee`, `Technician`, `TeamLead`, `Administrator`, `ITManager`) |
| **Incident Lifecycle** | Full CRUD and status transitions | `Incident` aggregate root governed by the **State Pattern** (9 distinct states) |
| **Priority Calculation** | Multi-factor prioritization | Matrix Strategy ($\text{Impact} \times \text{Urgency} \times \text{Criticality}$) and Weighted Formula Strategy |
| **Assignment Strategies** | $\ge 2$ dynamic strategies | **4 strategies**: Skill-Based, Load-Balanced, Round-Robin, and Smart Composite Scoring |
| **SLA Management** | Response/Resolution tracking, breach alerts | Dynamic timers, service overrides, SLA pause on `PENDING_INFO`, background SLA monitor |
| **Notification Engine** | $\ge 2$ communication channels | **3 channels**: In-app WebSocket (STOMP), Email (SMTP / Mailpit), SMS (Adapter pattern) |
| **Audit Trail** | Complete historical tracking | Immutable `AuditEntry` stream captured transactionally via the **Observer Pattern** |
| **Search & Filtering** | $\ge 5$ operational filters | **8+ criteria**: text query, status, priority, category, service, assignee, date range, SLA state |
| **Intelligent Features** | $\ge 2$ intelligent capabilities | **All 7 features implemented**: Duplicate Detection, Incident Correlation, Auto-Escalation, Smart Assignment, SLA Breach Prediction, Auto-Classification, Solution Recommender |
| **Persistence** | Relational database with schema evolution | PostgreSQL, Flyway versioned migrations, Hibernate schema validation |
| **Security Standards** | Secure authentication and RBAC | BCrypt hashing, stateless JWT, method security, lockout protection, RFC 7807 problem details |

---

## Object-Oriented Programming Principles

The platform serves as a reference implementation of core OOP tenets:

- **Encapsulation**: State fields inside `Incident`, `User`, and `SlaRecord` are private. Status transitions are exclusively handled through domain methods; setters that bypass business invariants are prohibited. Password hashes are isolated within entity boundaries.
- **Abstraction**: High-level components depend on abstract contracts (`IncidentState`, `AssignmentStrategy`, `PriorityStrategy`, `NotificationChannel`, `EmailGateway`, `SmsGateway`) rather than concrete implementations.
- **Inheritance ("Is-A")**: Strict behavioral specialization across the user domain:
  - `User` $\rightarrow$ `Employee`
  - `User` $\rightarrow$ `Technician` $\rightarrow$ `TeamLead`
  - `User` $\rightarrow$ `Administrator` $\rightarrow$ `ITManager`
  - `IncidentState` $\rightarrow$ `ActiveWorkState` $\rightarrow$ `AssignedState` / `InvestigatingState`
- **Polymorphism**: Dynamic dispatch across polymorphic interfaces, including `strategy.assign()`, `state.resolve()`, `channel.deliver()`, and `exception.getErrorCode()`.
- **Composition ("Has-A")**: An `Incident` is composed of its `SlaRecord`, `Resolution`, `Feedback`, and associated `IncidentNote` entities.
- **Exception Hierarchy**: Structured domain exceptions extending `ItsmException` mapped through `GlobalExceptionHandler` to RFC 7807 Problem Details.
- **Collections & Generics**: Specialized data structures utilized for operational efficiency:
  - `EnumMap` and `EnumSet` for constant-time state and permission lookups
  - `PriorityQueue` for ranking technician assignment scores
  - `ArrayDeque` for service dependency graph traversals (BFS)
  - `ConcurrentHashMap` and `AtomicInteger` for thread-safe round-robin distribution
  - `StrategyRegistry<T extends NamedStrategy>` for generic runtime strategy resolution

---

## Design Patterns Implemented (8 GoF Patterns)

| Design Pattern | Problem Solved in I-ITSM | Implementation Classes |
|---|---|---|
| **State** | Manages incident status transitions, enforcing strict lifecycle rules and disallowing illegal operations. | `IncidentState`, `NewState`, `AssignedState`, `InvestigatingState`, `PendingInfoState`, `ResolvedState`, `ClosedState` |
| **Strategy** | Enables runtime interchangeability of priority assessment and technician assignment algorithms. | `PriorityStrategy` (`Matrix`, `Weighted`), `AssignmentStrategy` (`Skill`, `Load`, `RoundRobin`, `Smart`) |
| **Observer** | Decouples domain events from cross-cutting concerns (audit logging, email dispatch, WebSocket toasts). | `IncidentEventPublisher`, `IncidentEventListener`, `InProcessEventBus` |
| **Factory** | Centralizes instantiation of complex aggregates and ensures initialization with valid default states. | `UserFactory`, `IncidentFactory`, `NotificationChannelFactory`, `IncidentStateFactory` |
| **Adapter** | Bridges external notification protocols (SMTP email and third-party SMS gateways) to unified internal interfaces. | `SmtpEmailGateway`, `LoggingEmailGateway`, `SimulatedSmsGatewayAdapter` |
| **Chain of Responsibility**| Routes unacknowledged or breached incidents through escalation levels until handled. | `EscalationHandler`, `TeamLeadEscalationHandler`, `AdminEscalationHandler`, `ManagerEscalationHandler` |
| **Template Method** | Defines the invariant ranking skeleton for assignment while letting concrete strategies specify scoring formulas. | `AbstractScoringAssignmentStrategy.rank()` |
| **Repository** | Abstracts database access and query construction behind clean domain collection semantics. | `IncidentRepository`, `UserRepository`, `AuditEntryRepository`, `IncidentSearchCriteria` |

---

## Project Structure

```
OOP-Project/
├── backend/
│   ├── src/main/java/com/itsm/
│   │   ├── config/             # Spring & application configuration
│   │   ├── domain/             # Entities: User, Incident, SLA, Audit, Catalog
│   │   ├── engine/             # Core engines: Assignment, Priority, SLA, Intelligence
│   │   ├── events/             # Observer pattern: Event types, bus, listeners
│   │   ├── exception/          # Custom exception hierarchy & RFC 7807 handler
│   │   ├── notification/       # Multi-channel notification adapters & dispatcher
│   │   ├── repository/         # Spring Data JPA repositories & specifications
│   │   ├── security/           # JWT filter, user principal, security configuration
│   │   ├── service/            # Application services & business transactions
│   │   └── web/                # REST API controllers & DTO mappings
│   ├── src/main/resources/
│   │   ├── db/migration/       # Flyway database schema migrations
│   │   └── application.yml     # Production & environment configurations
│   └── src/test/java/          # Unit, integration, and architecture test suites
├── frontend/
│   ├── src/
│   │   ├── api/                # HTTP & WebSocket API client modules
│   │   ├── auth/               # JWT authentication context & route guards
│   │   ├── components/         # Reusable UI primitives & layout elements
│   │   ├── features/           # Domain views: Incidents, Queue, Intelligence, Admin
│   │   ├── hooks/              # Custom hooks: real-time subscriptions, theme toggle
│   │   └── pages/              # Primary route views (Login, Register, Dashboard)
│   └── index.html              # HTML shell
├── docs/                       # Formal engineering specifications & reports
│   ├── REQUIREMENTS.md         # Problem analysis, business rules & functional specs
│   ├── DESIGN.md               # UML architecture, ER diagrams & algorithmic models
│   ├── TECHNOLOGY.md           # Technical stack justification & design tradeoffs
│   └── TEST_CASES.md           # Comprehensive test catalog mapped to test suites
└── docker-compose.yml          # Container orchestration specification
```

---

## Test Execution & Verification

### Backend Verification
The backend test suite covers unit logic, strategy mathematics, state transitions, repository queries, and MockMvc REST endpoints:

```bash
cd backend
./mvnw clean verify
```
*Generates JaCoCo instruction and branch coverage reports at `backend/target/site/jacoco/index.html`.*

### Frontend Verification
The frontend test suite validates form validation, password visibility toggling, accessible badge rendering, and utility calculations:

```bash
cd frontend
npm test -- --run
```

---

## Documentation Index

Comprehensive documentation covering requirements, system design, technology trade-offs, and verification test cases is located in the `docs/` folder:

- **[Requirements Specification (`docs/REQUIREMENTS.md`)](docs/REQUIREMENTS.md)**: Stakeholder identification, functional and non-functional requirements, business rules, and constraints.
- **[Architecture & Design Document (`docs/DESIGN.md`)](docs/DESIGN.md)**: Architectural diagrams, complete UML models (Use Case, Class, State Machine, Sequence), ER schema, and algorithmic specifications.
- **[Technology Selection Report (`docs/TECHNOLOGY.md`)](docs/TECHNOLOGY.md)**: Detailed technical justification, evaluation of alternatives, and evaluation question responses.
- **[Test Suite Catalog (`docs/TEST_CASES.md`)](docs/TEST_CASES.md)**: Documented unit, integration, and security test cases cross-referenced with automated test classes.
