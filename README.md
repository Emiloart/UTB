# UTB

## Ultra Hybrid Brain

UTB is an AI-native intelligence platform designed to continuously research, monitor, connect, and synthesize information into usable intelligence.

It is not a note-taking application with an AI assistant.

UTB is being built as a persistent research and intelligence system: a private AI analyst team that continuously watches the information environments a user or organization cares about, detects meaningful changes, builds relationships between facts, and produces intelligence without requiring a prompt for every investigation.

## The problem

Most knowledge tools are fundamentally reactive.

You open a workspace, ask a question, search for information, read the results, and decide what matters.

That model breaks down when the information environment is continuously changing.

A serious analyst may need to monitor:

- new research papers
- GitHub repositories and technical changes
- companies and products
- markets and economic indicators
- regulatory developments
- news and public sources
- emerging technologies
- people, organizations, and ecosystems
- relationships between seemingly unrelated events

UTB is designed around the opposite model:

> **The system keeps researching after the user closes the application.**

## Product model

```text
                         UTB
                          │
          ┌───────────────┼────────────────┐
          │               │                │
       Discover         Monitor          Reason
          │               │                │
          └───────────────┼────────────────┘
                          ↓
                 Knowledge Graph
                          ↓
                 Signal Detection
                          ↓
                Intelligence Layer
                          ↓
        ┌─────────────────┼─────────────────┐
        │                 │                 │
     Dashboard         Reports           Alerts
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                 APIs / Integrations
```

The core loop is:

**observe → normalize → understand → connect → detect → reason → report → learn**

## What UTB does

### Continuous research

UTB continuously ingests information from configured sources and converts it into structured research objects.

### Source monitoring

The platform can monitor sources such as:

- research repositories
- GitHub
- technical documentation
- news and public web sources
- market data
- regulatory sources
- enterprise data
- user-provided documents

### Multi-agent analysis

Specialized intelligence workers handle different forms of analysis.

Examples:

- Research Agent
- Code Intelligence Agent
- Market Agent
- Trend Agent
- Entity Resolution Agent
- Report Agent

Agents are workers inside a durable workflow system, not independent chatbots.

### Knowledge graph

UTB maintains relationships between:

- people
- organizations
- technologies
- papers
- repositories
- products
- markets
- events
- concepts

This allows the system to reason over connections rather than treating every document as an isolated blob of text.

### Signal detection

The objective is not merely to summarize what happened.

UTB should identify:

- emerging themes
- unusual activity
- accelerating research areas
- technology adoption shifts
- meaningful repository changes
- relationships between events
- changes in previously established assumptions

### Intelligence delivery

The system can produce:

- daily intelligence briefs
- weekly research reports
- event-driven alerts
- topic dashboards
- ecosystem maps
- source-specific monitoring
- API-accessible intelligence

## Architecture

UTB is intentionally polyglot.

| Layer | Technology |
| --- | --- |
| Web | Next.js, TypeScript |
| Mobile | Expo / React Native, TypeScript |
| Edge | Cloudflare |
| Core APIs | FastAPI |
| Infrastructure services | Go |
| AI / agent runtime | Python |
| Durable workflows | Temporal |
| Event streaming | Apache Kafka / Amazon MSK |
| Transactional database | PostgreSQL / Aurora PostgreSQL |
| Vector retrieval | pgvector |
| Search | OpenSearch |
| Knowledge graph | Neo4j |
| Analytics / OLAP | ClickHouse |
| Cache | Redis / Amazon ElastiCache |
| Object storage | Amazon S3 |
| Identity | OIDC / SAML / WebAuthn through a dedicated identity provider |
| Compute | Kubernetes / Amazon EKS |
| Infrastructure as code | Terraform |
| Deployment | GitHub Actions + Argo CD |
| Observability | OpenTelemetry |
| Edge security | Cloudflare WAF / bot protection |

The architecture is deliberately split by workload instead of forcing every problem into one database or one programming language.

## System architecture

```text
                         INTERNET
                            │
                      Cloudflare Edge
                  CDN / WAF / Rate Limits
                            │
             ┌──────────────┴──────────────┐
             │                             │
          Web App                     API Gateway
        Next.js/TS                   Auth / Routing
             │                             │
             └──────────────┬──────────────┘
                            │
                    UTB Control Plane
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       Workspace          Research          Agent
       Services           Services          Runtime
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                         Temporal
                            │
                    Durable Workflows
                            │
                    Kafka Event Bus
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      Connectors        Enrichment        Detection
       / Sources          Workers           Workers
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                 ┌──────────┼──────────┐
                 │          │          │
             PostgreSQL  OpenSearch  Neo4j
             + pgvector               │
                 │          │          │
                 └──────────┼──────────┘
                            │
                       ClickHouse
                            │
                     Intelligence API
                            │
              ┌─────────────┼─────────────┐
              │             │             │
          Dashboards      Reports        Alerts
```

## Data architecture

UTB deliberately uses multiple storage systems because its workloads are fundamentally different.

### PostgreSQL

System of record for:

- accounts
- organizations
- workspaces
- permissions
- subscriptions
- connectors
- jobs
- configuration
- report definitions
- audit metadata

### pgvector

Semantic storage for tightly relational memory:

- user/workspace memory
- compact embeddings
- semantic references
- retrieval metadata

### OpenSearch

Large-scale retrieval and search:

- documents
- chunks
- source content
- full-text indexes
- hybrid vector/keyword retrieval

### Neo4j

Relationship intelligence:

- entity relationships
- source provenance
- technology ecosystems
- organization relationships
- research relationships
- dependency structures

### ClickHouse

Analytical workloads:

- event analytics
- trend measurements
- ingestion statistics
- usage analytics
- time-series intelligence
- aggregate reporting

### S3

Durable object storage:

- original source documents
- normalized artifacts
- generated reports
- exports
- raw ingestion payloads
- data lake objects

## Agent architecture

UTB agents are not the system of record.

Agents operate through explicit tools and durable workflows.

```text
                 Temporal Workflow
                        │
              ┌─────────┴─────────┐
              │                   │
         Planning Agent       Retrieval
              │                   │
              └─────────┬─────────┘
                        ↓
                  Tool Execution
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      Search          Graph            Code
        │               │                │
      Source          Entity           GitHub
      Retrieval       Lookup           Analysis
        │               │                │
        └───────────────┼────────────────┘
                        ↓
                  Evidence Set
                        ↓
                  Reasoning Pass
                        ↓
                 Confidence Check
                        ↓
                    Synthesis
                        ↓
                  Stored Insight
```

Every important conclusion should be traceable to its supporting evidence.

UTB therefore treats **provenance** as a first-class data primitive.

## Model architecture

UTB should never be coupled directly to one model provider.

All model access passes through an internal model gateway.

The gateway handles:

- provider selection
- model routing
- fallbacks
- token budgets
- latency budgets
- cost controls
- structured output
- prompt/version management
- model evaluation
- safety controls
- telemetry

The application should request capabilities rather than hard-code a vendor model.

For example:

```text
REASONING_HIGH
REASONING_FAST
CLASSIFICATION
EXTRACTION
EMBEDDING
RERANKING
VISION
```

This allows UTB to change model providers without rewriting the product.

## Global architecture

UTB is designed for global adoption from the beginning.

The intended topology is:

```text
                 Global Users
                      │
                Cloudflare Edge
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Europe       Americas      APAC
       Region        Region       Region
          │           │           │
          └───────────┼───────────┘
                      │
                Global Data Layer
                      │
              Regional Read Paths
                      │
               Controlled Writes
```

The system should distinguish:

- global control-plane configuration
- regional user data
- tenant-specific data residency
- public-source data
- private enterprise data

Global deployment does not mean every dataset is replicated everywhere.

## Security principles

UTB is an intelligence platform. Its security model must assume that the data itself can be strategically sensitive.

Core principles:

1. Tenant isolation by default.
2. Least-privilege service identities.
3. Encryption in transit and at rest.
4. Centralized secrets management.
5. Immutable audit trails for sensitive actions.
6. Explicit tool permissions for agents.
7. Human approval for high-impact actions.
8. Data minimization before external model calls.
9. Source provenance for generated intelligence.
10. Regional data controls where required.

The agent runtime must not receive unrestricted access to the entire platform.

A tool is granted because a workflow requires it, not because the agent is an administrator.

## Repository philosophy

UTB should evolve toward a modular monorepo rather than an uncontrolled collection of services.

A target structure:

```text
utb/
├── apps/
│   ├── web/
│   ├── mobile/
│   └── admin/
│
├── services/
│   ├── api/
│   ├── ingestion/
│   ├── research/
│   ├── intelligence/
│   ├── agents/
│   ├── search/
│   ├── graph/
│   └── notifications/
│
├── packages/
│   ├── contracts/
│   ├── auth/
│   ├── database/
│   ├── events/
│   ├── ai/
│   ├── observability/
│   └── sdk/
│
├── infrastructure/
│   ├── terraform/
│   ├── kubernetes/
│   └── argocd/
│
├── docs/
│   ├── BUILD.md
│   ├── ARCHITECTURE.md
│   ├── DATA_MODEL.md
│   ├── SECURITY.md
│   └── ADRs/
│
└── .github/
    └── workflows/
```

The exact service boundaries should emerge from workload boundaries. Do not create microservices merely to make a diagram look sophisticated.

## Development principles

UTB follows several engineering rules.

### Evidence over prose

An intelligence statement without provenance is incomplete.

### Durable workflows over fragile automation

Long-running research belongs in Temporal, not a chain of cron jobs.

### Structured data over prompt memory

Important state belongs in databases, graphs, events, or object storage.

### Provider independence

Models are replaceable infrastructure.

### Explicit uncertainty

UTB must distinguish:

- observed fact
- extracted fact
- inferred relationship
- model-generated hypothesis
- unresolved uncertainty

### Human control

Autonomy applies to research and analysis. High-impact external actions require explicit authorization.

### Global by design

Regional deployment, data residency, localization, and enterprise identity are architecture concerns, not late-stage patches.

## Current status

UTB is at the architecture/foundation stage.

The repository is being established around the long-term system design before implementation is allowed to fragment into disconnected features.

The first implementation phase should prove the complete intelligence loop with a narrow source set:

```text
Source
  ↓
Ingestion
  ↓
Normalization
  ↓
Entity extraction
  ↓
Storage
  ↓
Retrieval
  ↓
Agent analysis
  ↓
Evidence-backed insight
  ↓
Report / alert
```

Once this loop is reliable, additional sources and intelligence domains can be added without changing the fundamental architecture.

## Long-term direction

UTB is intended to become an intelligence infrastructure platform rather than another AI chat application.

The product surface is the dashboard.

The real product is the continuously evolving intelligence system underneath it:

**data → memory → relationships → signals → reasoning → intelligence**

That distinction governs the architecture.

## License

License and contribution policy will be established as the project moves toward public collaboration.
