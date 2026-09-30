# UTB Build Specification

## 1. Purpose

This document defines how UTB is intended to be built and the architectural decisions that should remain stable as implementation begins.

UTB is an AI-native research and intelligence platform. Its defining property is continuous operation.

A user should be able to define an intelligence domain and then allow UTB to:

1. discover relevant sources;
2. monitor them continuously;
3. ingest changes;
4. normalize and enrich information;
5. resolve entities;
6. update a knowledge graph;
7. retrieve supporting evidence;
8. run specialized analysis;
9. detect meaningful signals;
10. produce reports and alerts;
11. retain provenance and uncertainty.

The system should remain useful even when the user is not actively interacting with it.

---

## 2. Architectural invariants

These are decisions that should not be casually changed because a particular framework becomes fashionable.

### 2.1 Workflows are durable

Long-running work is modeled as a durable workflow.

Temporal is the workflow authority.

### 2.2 Events are first-class

Services communicate asynchronously through versioned events where work does not require synchronous request/response semantics.

Kafka is the principal event backbone.

### 2.3 Databases have explicit responsibilities

No database is allowed to become the accidental home of unrelated workloads.

- PostgreSQL: transactional truth
- OpenSearch: retrieval/search
- Neo4j: relationships
- ClickHouse: analytical aggregation
- S3: durable objects
- Redis: ephemeral acceleration

### 2.4 Agents do not own business state

Agents can reason and request tools.

They do not become the authoritative source of user, billing, permission, or research state.

### 2.5 Every intelligence claim has provenance

A report should be able to answer:

- What source produced this?
- When was it observed?
- What transformation occurred?
- Which model or algorithm analyzed it?
- What evidence supports the conclusion?
- How certain is the system?

---

## 3. Major system domains

### Identity

Responsible for:

- users
- organizations
- workspaces
- sessions
- enterprise identity
- roles

### Source Management

Responsible for:

- source registration
- connector configuration
- credentials
- polling schedules
- webhook registration
- source health

### Ingestion

Responsible for:

- fetching
- webhook reception
- deduplication
- normalization
- content extraction
- raw artifact preservation

### Research

Responsible for:

- documents
- citations
- research topics
- collections
- retrieval
- evidence sets

### Knowledge

Responsible for:

- entities
- relationships
- entity resolution
- graph construction
- provenance

### Intelligence

Responsible for:

- signal detection
- trend analysis
- anomaly detection
- scoring
- synthesis

### Agents

Responsible for:

- planning
- tool selection
- reasoning
- synthesis
- report generation

### Delivery

Responsible for:

- dashboards
- reports
- notifications
- alerts
- APIs
- exports

---

## 4. Request architecture

Interactive requests follow:

```text
Client
  ↓
Cloudflare
  ↓
API
  ↓
Authorization
  ↓
Domain Service
  ↓
PostgreSQL / Search / Graph
  ↓
Response
```

Interactive APIs must remain fast.

If an operation may take more than a normal request lifecycle, create a durable job and return a job/workflow identifier.

---

## 5. Research pipeline

The fundamental research pipeline is:

```text
Connector
  ↓
Raw Artifact
  ↓
Canonical Document
  ↓
Content Extraction
  ↓
Chunking
  ↓
Embedding
  ↓
Entity Extraction
  ↓
Entity Resolution
  ↓
Graph Update
  ↓
Search Index
  ↓
Signal Processing
```

Each stage should be independently observable and retryable.

Failures should not require replaying the entire pipeline.

---

## 6. Event model

Events should be immutable and versioned.

Example:

```json
{
  "event_id": "uuid",
  "event_type": "source.document.observed",
  "schema_version": 1,
  "occurred_at": "2026-09-30T00:00:00Z",
  "tenant_id": "uuid",
  "source_id": "uuid",
  "correlation_id": "uuid",
  "payload": {}
}
```

Recommended event families:

```text
source.*
document.*
entity.*
graph.*
research.*
signal.*
workflow.*
report.*
notification.*
```

Consumers must be idempotent.

At-least-once delivery should therefore be assumed.

---

## 7. Source connector contract

Every connector should expose a consistent internal interface.

Conceptually:

```text
Connector
├── authenticate()
├── health()
├── discover()
├── fetch()
├── normalize()
├── checkpoint()
└── subscribe()
```

Not every source supports every operation.

For example:

- GitHub supports webhooks.
- RSS may support polling only.
- Some APIs support incremental cursors.
- Some web sources require scheduled crawling.

The connector layer absorbs these differences.

The rest of UTB should receive a canonical event model.

---

## 8. GitHub intelligence

GitHub is an important initial source because repository activity contains structured technical signals.

Monitor:

- commits
- releases
- pull requests
- issues
- contributors
- dependency changes
- repository metadata
- documentation changes

Potential intelligence:

- activity acceleration
- contributor concentration
- architectural changes
- dependency migration
- emerging libraries
- project abandonment
- ecosystem relationships

A raw commit should not immediately become an intelligence signal.

The system should accumulate evidence over time.

---

## 9. Research-paper intelligence

For research sources:

```text
Paper
 ↓
Metadata extraction
 ↓
Full text where legally available
 ↓
Section-aware parsing
 ↓
Claims / methods / datasets / citations
 ↓
Entity resolution
 ↓
Embedding + search index
 ↓
Knowledge graph
 ↓
Trend analysis
```

The system should distinguish the paper's claims from UTB's interpretation.

Generated summaries must retain source references.

---

## 10. Knowledge graph

The graph is not a decorative visualization.

It is a computational layer.

Example:

```text
Researcher
   │
   ├── authored ──→ Paper
   │                  │
   │                  ├── introduces ──→ Technique
   │                  │
   │                  └── cites ───────→ Paper
   │
   └── contributes ─→ Repository
                         │
                         └── depends_on ─→ Library
```

The graph should maintain:

- node identity
- relationship type
- confidence
- source
- observed timestamp
- valid-from / valid-to where appropriate

This allows UTB to distinguish a current relationship from an obsolete one.

---

## 11. Retrieval architecture

Use hybrid retrieval.

```text
User / Agent Query
       │
       ├──────────────→ Keyword Search
       │
       ├──────────────→ Vector Search
       │
       └──────────────→ Graph Traversal
                              │
                              ↓
                       Candidate Evidence
                              │
                              ↓
                           Rerank
                              │
                              ↓
                       Evidence Set
```

A single vector similarity score should never be treated as sufficient evidence for important intelligence.

---

## 12. Agent execution

A typical research workflow:

```text
Trigger
  ↓
Create workflow
  ↓
Load research objective
  ↓
Plan investigation
  ↓
Retrieve evidence
  ↓
Call specialized tools
  ↓
Validate evidence
  ↓
Update graph
  ↓
Generate findings
  ↓
Check confidence / contradictions
  ↓
Write report
  ↓
Persist provenance
  ↓
Deliver
```

The workflow should be resumable at every meaningful stage.

---

## 13. Model gateway

All LLM requests go through:

```text
Agent
  ↓
UTB AI Gateway
  ↓
Policy
  ├── task type
  ├── model capability
  ├── cost budget
  ├── latency budget
  ├── privacy policy
  └── provider availability
  ↓
Model Provider
```

The gateway records:

- model
- provider
- input/output tokens
- latency
- cost
- request purpose
- workflow ID
- evaluation metadata

Prompts and structured schemas should be versioned.

---

## 14. AI evaluation

UTB should maintain evaluation datasets before aggressively expanding autonomous behavior.

Evaluate:

- retrieval precision
- citation correctness
- entity resolution
- factuality
- contradiction detection
- report completeness
- signal precision
- false-positive rate
- agent tool selection
- workflow recovery

Every major model or prompt change should run regression evaluations.

The platform should optimize for **useful intelligence**, not impressive prose.

---

## 15. Multi-tenancy

Every tenant-owned object should have a tenant boundary.

Conceptually:

```text
Organization
  └── Workspace
       ├── Sources
       ├── Documents
       ├── Entities
       ├── Research
       ├── Reports
       └── Agents
```

Authorization must be enforced server-side.

Tenant identifiers should propagate through:

- HTTP requests
- workflow payloads
- Kafka events
- database records
- object keys
- logs
- traces

A missing tenant context is a security defect.

---

## 16. Security boundaries

The most sensitive boundary is:

```text
User / Tenant Data
        ↓
UTB Processing
        ↓
External Model Provider
```

Before information crosses the external model boundary:

- classify sensitivity
- apply tenant policy
- redact where required
- record the destination
- record the model
- enforce allowed-data policy

Enterprise customers should eventually be able to select:

- provider
- region
- retention
- encryption key
- private networking model

---

## 17. Observability

Every request and workflow should carry:

- request ID
- trace ID
- tenant ID
- workflow ID
- source ID where relevant

Measure:

### Platform

- API latency
- error rate
- CPU/memory
- queue lag
- Kafka consumer lag
- database latency

### Research

- ingestion latency
- source freshness
- processing failure rate
- duplicate rate
- extraction success

### Intelligence

- retrieval precision
- signal volume
- false-positive rate
- report generation latency
- evidence coverage

### AI

- model latency
- token usage
- cost per workflow
- tool-call failures
- model error rate

---

## 18. Deployment environments

Minimum environments:

```text
local
  ↓
development
  ↓
staging
  ↓
production
```

Production infrastructure must never depend on developer machines.

Secrets must be injected through managed secret infrastructure.

Infrastructure changes must be reviewed through code.

---

## 19. Repository implementation strategy

Start as a modular monorepo.

Do not deploy every package as an independent service on day one.

Initial implementation can be:

```text
apps/web
services/api
services/worker
packages/contracts
packages/database
packages/ai
packages/events
packages/observability
```

As workloads become independently scalable, extract:

```text
ingestion
search
graph
intelligence
notifications
```

This preserves architectural boundaries without paying premature microservice complexity.

---

## 20. First implementation milestone

The first milestone is not the dashboard.

It is an end-to-end intelligence loop.

### Vertical slice

Implement one source and one research objective.

Example:

```text
GitHub Repository
      ↓
Webhook / Poller
      ↓
Kafka
      ↓
Normalizer
      ↓
Postgres + S3
      ↓
OpenSearch
      ↓
Entity Extraction
      ↓
Neo4j
      ↓
Research Agent
      ↓
Evidence-backed Finding
      ↓
Report
```

If this works reliably, the architecture has been validated.

---

## 21. Implementation phases

### Phase 0 — Foundation

- repository structure
- TypeScript contracts
- Python service foundation
- Go connector foundation
- local development environment
- CI
- environment configuration
- observability
- database migrations

### Phase 1 — First intelligence loop

- one source connector
- ingestion
- document normalization
- storage
- retrieval
- one agent workflow
- provenance
- basic report generation

### Phase 2 — Knowledge layer

- entity extraction
- entity resolution
- Neo4j
- relationship provenance
- graph retrieval

### Phase 3 — Continuous intelligence

- scheduled research
- source monitoring
- signal detection
- daily reports
- alerts
- workflow retries
- source health

### Phase 4 — Multi-source intelligence

- research papers
- GitHub
- news/public web
- technical documentation
- market feeds
- user documents

### Phase 5 — Enterprise architecture

- organizations
- SSO
- SCIM
- advanced RBAC/ABAC
- data residency
- BYOK
- private networking
- dedicated deployments

### Phase 6 — Intelligence platform

- external APIs
- SDKs
- integrations
- intelligence feeds
- domain-specific agents
- partner ecosystem

---

## 22. What should not be built yet

Avoid:

- dozens of agents
- elaborate graph visualizations
- a marketplace
- autonomous external actions
- every imaginable connector
- multi-cloud deployment
- custom model training
- premature active-active writes
- a huge microservice fleet

First prove that UTB can repeatedly turn **new information into trustworthy intelligence**.

---

## 23. Definition of success

The first serious UTB milestone is reached when a user can define:

> "Monitor this domain and tell me what materially changed."

UTB should then independently:

1. discover relevant information;
2. determine what is new;
3. connect it to existing knowledge;
4. identify why it matters;
5. provide evidence;
6. state uncertainty;
7. notify the user;
8. remember the result for future investigations.

That is the core product.

Everything else is expansion around this loop.

---

## 24. Architectural north star

UTB should eventually operate as:

```text
                 WORLD
                   │
             DATA / EVENTS
                   │
             INGESTION LAYER
                   │
             KNOWLEDGE LAYER
                   │
        ┌──────────┴──────────┐
        │                     │
   MEMORY / GRAPH        SEARCH / RETRIEVAL
        │                     │
        └──────────┬──────────┘
                   │
              AGENT SYSTEM
                   │
            SIGNAL ENGINE
                   │
          INTELLIGENCE LAYER
                   │
        ┌──────────┼──────────┐
        │          │          │
     REPORTS     ALERTS      APIs
        │          │          │
        └──────────┼──────────┘
                   │
                 USERS
```

The dashboard is the visible surface.

The continuously updated intelligence substrate is UTB.
