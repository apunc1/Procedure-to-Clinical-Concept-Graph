# Procedure-to-Clinical-Concept Graph

## Detailed Product & Development Plan

## 1. Executive Summary

The **Procedure-to-Clinical-Concept Graph** is a healthcare intelligence product that connects procedure codes such as CPT/HCPCS to the clinical concepts, diagnoses, conditions, specialties, providers, settings, and downstream utilization patterns associated with them.

A procedure code by itself is primarily a billing/coding representation.

The graph attempts to answer a broader question:

> **"What clinical concepts and real-world healthcare activity are associated with this procedure?"**

Instead of analyzing CPT/HCPCS codes as isolated rows, the system represents healthcare relationships as a network:

```text
Procedure
    ↓
Clinical Concept
    ↓
Diagnosis
    ↓
Patient / Population
    ↓
Provider / Specialty
    ↓
Place of Service
    ↓
Coverage / Authorization
    ↓
Denial / Utilization
```

The product provides an interactive way to explore these relationships and identify patterns that would be difficult to see in a traditional relational table or dashboard.

---

# 2. Product Concept

### Product Name

**Procedure-to-Clinical-Concept Graph**

### Product Category

**Healthcare Clinical & Procedure Intelligence**

### Tagline

> **Connect the code to the clinical story.**

### Core Product Question

> **"What clinical concepts, diagnoses, providers, settings, and healthcare activities are connected to this procedure?"**

---

# 3. Why a Graph?

Traditional healthcare analytics often looks like this:

```text
CPT → claims → diagnosis
```

and requires users to manually join tables.

A graph represents the relationships directly.

For example:

```text
                ┌── ICD-10 A
                │
CPT 12345 ──────┼── ICD-10 B
                │
                └── ICD-10 C
                     │
                     ↓
                 Condition
                     │
              ┌──────┴──────┐
              ↓             ↓
          Provider A     Provider B
              │             │
              ↓             ↓
           Office        Hospital
```

This makes it possible to investigate relationships rather than simply retrieve records.

---

# 4. The Problem

Procedure-code intelligence frequently stops at:

* Code description
* Procedure category
* Utilization
* Price
* Claim count

But users often need to understand:

* What conditions is this procedure associated with?
* What diagnoses commonly occur with it?
* Which specialties use it?
* Where is it performed?
* Which procedures are clinically similar?
* Which diagnoses are unusually associated with it?
* Which procedures tend to occur before or after it?
* Which providers use it?
* Which coverage policies relate to it?
* Which authorization requirements may apply?
* Which denial patterns are associated with it?

These relationships are naturally represented as a graph.

---

# 5. Product Vision

The long-term vision is a **Healthcare Procedure Knowledge Graph**.

```text
                  PROCEDURE
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Diagnosis      Specialty       POS
        │             │             │
        ↓             ↓             ↓
 Clinical Concept  Provider      Care Setting
        │
        ↓
    Population
        │
        ↓
 Related Procedures
        │
        ├──────── Coverage
        │
        ├──────── Authorization
        │
        └──────── Denials
```

The graph eventually becomes a reusable intelligence layer for the other products in your portfolio.

---

# 6. Target Users

### Healthcare Analysts

* Claims analysts
* Data analysts
* Clinical analysts
* Utilization analysts
* BI analysts

### Product Teams

* Healthcare product managers
* Benefits analysts
* Network teams
* Operations teams

### Clinical / Utilization Teams

* Utilization management
* Prior authorization
* Medical policy analysts
* Care-management analysts

### Data Science / AI Teams

* Data scientists
* ML engineers
* AI teams
* Knowledge-management teams

For your portfolio, the primary persona should be:

> **Healthcare analyst investigating how a procedure is used across clinical and operational contexts.**

---

# 7. Core Graph Entities

The first version should not try to model every healthcare concept.

Start with a manageable set of nodes.

### Procedure

```text
CPT
HCPCS
```

### Diagnosis

```text
ICD-10-CM
```

### Clinical Concept

Higher-level concepts derived from diagnosis hierarchy.

Examples:

```text
Disease
Condition
Symptom
Anatomical system
Disease category
```

### Provider

```text
NPI
Provider specialty
Provider taxonomy
```

### Place of Service

Examples:

```text
Office
Hospital outpatient
Inpatient
ASC
Laboratory
DME
```

### Coverage Policy

```text
NCD
LCD
Article
```

### Authorization Rule

```text
PA requirement
Documentation requirement
```

### Denial

```text
Denial category
Denial code
```

---

# 8. Graph Relationships

The relationships are as important as the nodes.

Examples:

```text
PROCEDURE
    ├── ASSOCIATED_WITH → DIAGNOSIS
    ├── USED_FOR → CLINICAL_CONCEPT
    ├── PERFORMED_BY → PROVIDER
    ├── PERFORMED_AT → PLACE_OF_SERVICE
    ├── COVERED_BY → POLICY
    ├── SUBJECT_TO → AUTHORIZATION_RULE
    ├── ASSOCIATED_WITH → DENIAL_CATEGORY
    └── RELATED_TO → PROCEDURE
```

Diagnosis relationships:

```text
DIAGNOSIS
    ├── PART_OF → CLINICAL_CONCEPT
    ├── ASSOCIATED_WITH → PROCEDURE
    └── ASSOCIATED_WITH → OTHER_DIAGNOSIS
```

Provider relationships:

```text
PROVIDER
    ├── SPECIALIZES_IN → SPECIALTY
    ├── PERFORMS → PROCEDURE
    └── SERVES → CLINICAL_CONCEPT
```

---

# 9. Start With a Bipartite Graph

For the MVP, don't build the entire graph immediately.

Start with:

```text
Procedure ↔ Diagnosis
```

This is the foundational relationship.

For example:

```text
CPT 12345
    │
    ├── ICD-10 A
    ├── ICD-10 B
    ├── ICD-10 C
    └── ICD-10 D
```

Each relationship should have attributes.

```text
procedure_code
diagnosis_code
claim_count
unique_beneficiaries
unique_providers
first_seen
last_seen
```

This gives you a measurable graph rather than a purely conceptual one.

---

# 10. Relationship Strength

One of the most important design decisions is that an edge should contain evidence.

Example:

```text
CPT 12345
      │
      │  2,481 claims
      │  183 beneficiaries
      │  24 providers
      ↓
ICD-10 A
```

Instead of simply:

```text
CPT 12345 → ICD-10 A
```

You can calculate a relationship strength based on measurable claims evidence.

Potential metrics:

* Claim count
* Patient count
* Provider count
* Frequency
* Percentage of procedure claims associated with diagnosis
* Percentage of diagnosis claims involving procedure

Do **not** initially call this "clinical strength."

Call it something like:

> **Observed claims association**

This is an important distinction because claims data demonstrate coding/utilization relationships, not necessarily clinical causation.

---

# 11. Clinical Concept Layer

ICD-10 codes can be difficult for a nonclinical user to interpret.

Create a hierarchy:

```text
ICD-10
   ↓
ICD-10 Category
   ↓
Disease / Condition
   ↓
Clinical Concept
   ↓
Body System
```

For example:

```text
ICD-10 code
    ↓
Specific diagnosis
    ↓
Diagnosis category
    ↓
Condition
    ↓
Body system
```

This lets users move between detailed and high-level views.

---

# 12. Important Design Principle

The graph should distinguish **observed relationships** from **semantic relationships**.

### Observed relationship

Derived from claims:

> CPT 12345 frequently appears with ICD-10 A.

### Semantic relationship

Derived from a reference hierarchy:

> ICD-10 A belongs to clinical concept X.

### Policy relationship

Derived from policy:

> CPT 12345 appears in LCD Y.

### Inferred relationship

Produced by an ML/LLM process:

> CPT 12345 may be related to procedure Z based on observed utilization patterns.

These should never be presented as equivalent evidence.

---

# 13. Data Architecture

Use the same local architecture as your other portfolio projects.

```text
CMS / Reference Data
        ↓
Python ETL
        ↓
Parquet
        ↓
DuckDB / pandas
        ↓
Graph Construction
        ↓
Graph Analytics
        ↓
Evidence Layer
        ↓
ML / LLM
        ↓
Streamlit
```

You don't need a dedicated graph database for the first version.

---

# 14. Recommended Technology Stack

### Data

* Python
* pandas
* NumPy
* Parquet

### Querying

* DuckDB

### Graph

Start with:

* NetworkX

Potential later technologies:

* Neo4j
* Amazon Neptune
* another graph database

NetworkX is sufficient for a portfolio MVP.

### Visualization

* Plotly
* Network visualization library compatible with Streamlit
* potentially PyVis for interactive graph exploration

### Application

* Streamlit

### ML

* scikit-learn

### AI

* LLM API

---

# 15. Core Data Model

## `procedure_node`

```text
procedure_code
code_system
description
category
subcategory
effective_date
termination_date
```

## `diagnosis_node`

```text
icd10_code
description
category
subcategory
clinical_concept
body_system
```

## `provider_node`

```text
provider_id
taxonomy
specialty
state
```

## `procedure_diagnosis_edge`

```text
procedure_code
diagnosis_code
claim_count
beneficiary_count
provider_count
first_seen
last_seen
procedure_share
diagnosis_share
```

## `procedure_provider_edge`

```text
procedure_code
provider_id
claim_count
beneficiary_count
```

## `procedure_pos_edge`

```text
procedure_code
place_of_service
claim_count
beneficiary_count
```

---

# 16. Building the Graph

Conceptually:

```python
graph.add_node(
    "CPT:12345",
    type="procedure"
)

graph.add_node(
    "ICD10:A123",
    type="diagnosis"
)

graph.add_edge(
    "CPT:12345",
    "ICD10:A123",
    claim_count=2481
)
```

The important part is that the graph is backed by an analytical data model.

The graph should not become a separate source of truth.

---

# 17. Procedure Investigation Workflow

The primary user experience should be:

### Step 1

Select a CPT/HCPCS code.

### Step 2

Display the procedure profile.

### Step 3

Show connected clinical concepts.

### Step 4

Expand a concept.

### Step 5

Show associated diagnoses.

### Step 6

Show providers and specialties.

### Step 7

Show places of service.

### Step 8

Show related procedures.

### Step 9

Show coverage and authorization relationships.

### Step 10

Show associated denial patterns.

This creates a natural investigation workflow.

---

# 18. Streamlit Page 1 — Procedure Explorer

User selects:

```text
CPT/HCPCS
```

Display:

```text
Procedure
CPT 12345

Description
...

Historical claims
...

Providers
...

Clinical concepts
...
```

Then show the graph.

---

# 19. Streamlit Page 2 — Clinical Graph Explorer

Example:

```text
                  Diagnosis A
                       │
                       │
Provider A ─── CPT 12345 ─── Diagnosis B
                       │
                       │
                 Clinical Concept
                       │
                       ↓
                Related Procedure
```

Users should be able to click a node and expand its relationships.

This is the visual centerpiece of the application.

---

# 20. Streamlit Page 3 — Clinical Concept Explorer

Instead of starting with a procedure, start with a clinical concept.

Example:

```text
Clinical Concept
    ↓
Conditions
    ↓
Procedures
    ↓
Providers
    ↓
Places of Service
```

This supports the reverse question:

> "Given a clinical condition, what procedures are commonly observed?"

---

# 21. Streamlit Page 4 — Procedure Relationships

Show related procedures.

Possible relationships:

### Shared diagnoses

Two procedures frequently appear with the same diagnoses.

### Sequential relationships

Procedure A frequently occurs before Procedure B in the same patient's claims history.

### Provider overlap

The same providers perform both procedures.

### Site-of-service overlap

Both procedures commonly occur in the same settings.

This creates increasingly sophisticated graph relationships.

---

# 22. Procedure Similarity

A useful analytical feature is:

> **"Find procedures similar to this procedure."**

Possible similarity features:

* Shared diagnoses
* Shared providers
* Shared specialties
* Shared POS
* Shared clinical concepts
* Similar utilization patterns

For example:

```text
Selected Procedure
       ↓
Shared diagnosis profile
       +
Shared provider profile
       +
Shared POS profile
       ↓
Related procedures
```

This can be implemented initially with cosine similarity on procedure feature vectors.

---

# 23. Procedure Substitution Intelligence

This becomes an interesting extension.

If two procedures have similar clinical/use profiles:

```text
Procedure A
       ↕
Procedure B
```

the system can flag:

> "These procedures have overlapping observed diagnosis and provider profiles."

However, it should **not** conclude that one is clinically interchangeable with the other.

That requires clinical evidence beyond claims association.

Use terminology such as:

> **Observed utilization similarity**

rather than:

> "Clinical equivalent."

---

# 24. Temporal Relationships

Once the basic graph works, add time.

Instead of:

```text
A → B
```

you can analyze:

```text
A
 ↓
30 days
 ↓
B
```

Potential relationships:

* Procedure sequences
* Diagnosis before procedure
* Procedure after diagnosis
* Follow-up procedures
* Repeat procedures

For example:

```text
Diagnosis
   ↓
Procedure A
   ↓
Procedure B
   ↓
Follow-up procedure
```

This begins to connect the project to your interest in **patient journeys** and sequence/Markov analysis.

---

# 25. Provider Intelligence

Add providers to the graph.

```text
                 Procedure
                 /       \
                /         \
        Provider A       Provider B
           |                 |
       Specialty A       Specialty B
```

Users could investigate:

> "Which specialties perform this procedure?"

and:

> "Which procedures are commonly performed by providers who use this procedure?"

Again, these are utilization observations rather than judgments about appropriate care.

---

# 26. Coverage Intelligence

Connect the graph to CMS coverage information.

```text
Procedure
    ↓
LCD
    ↓
Diagnosis requirements
```

This allows users to explore:

> "Which clinical concepts are connected to procedures appearing in this coverage policy?"

It also creates a bridge into your **Prior Authorization Risk Engine**.

---

# 27. Denial Intelligence

Connect denial information:

```text
Procedure
     ↓
Diagnosis
     ↓
Denial Category
```

For example:

```text
CPT 12345
   │
   ├── Diagnosis A
   │
   ├── Diagnosis B
   │
   └── Authorization-related denials
```

This allows analysts to investigate whether certain procedure/diagnosis relationships coincide with particular denial patterns.

---

# 28. Machine Learning

ML should come **after the graph itself works**.

Potential ML applications:

### Procedure Similarity

Find procedures with similar relationship profiles.

### Community Detection

Identify clusters of:

* Procedures
* Diagnoses
* Providers
* Clinical concepts

### Anomaly Detection

Identify unusual procedure/diagnosis relationships.

Example:

```text
Procedure A
      ↓
Usually associated with
      ↓
Diagnosis cluster X

New relationship
      ↓
Diagnosis cluster Z
```

This becomes a potential investigation signal.

It should not automatically be interpreted as inappropriate coding.

---

# 29. Graph-Based Anomaly Detection

This could become one of the more advanced features.

Suppose:

```text
Procedure A
   ↓
99% of observed relationships
   ↓
Clinical Concept X
```

but a particular provider has:

```text
Procedure A
   ↓
Clinical Concept Y
```

The system could flag:

> **"This procedure/clinical-concept relationship differs from the observed pattern in the selected comparison population."**

The user then investigates.

This connects naturally to your **Denial Prevention Engine** and your existing interest in anomaly detection.

---

# 30. LLM Layer

The LLM should translate the graph into understandable language.

Example user question:

> "What clinical concepts are most strongly associated with this procedure?"

The application retrieves:

```text
Procedure
↓
Top diagnosis associations
↓
Clinical hierarchy
↓
Provider/POS evidence
```

The LLM then generates a summary.

Example:

> "In the selected claims population, this procedure is most frequently associated with diagnoses within three clinical concept groups. The strongest observed association is Concept A, followed by Concepts B and C. The procedure is primarily observed among providers in specialties X and Y."

The LLM should be explicitly instructed to describe **observed claims associations**, not clinical causation.

---

# 31. LLM Guardrails

```text
Use only retrieved graph evidence.

Do not infer causality.

Do not infer medical necessity.

Do not state that a procedure is clinically appropriate.

Do not claim two procedures are clinically interchangeable.

Do not invent clinical relationships.

Clearly distinguish:
- claims association
- reference hierarchy
- policy relationship
- model-derived similarity
```

This makes the AI component substantially more defensible.

---

# 32. Phase 1 — Graph Intelligence MVP

### Goal

Build a functioning Procedure → Diagnosis → Clinical Concept graph.

### Scope

1. Load CPT/HCPCS reference data.
2. Load ICD-10 reference data.
3. Load synthetic claims.
4. Aggregate procedure/diagnosis relationships.
5. Build graph with NetworkX.
6. Add relationship metrics.
7. Build interactive Streamlit explorer.

### No ML yet.

### No LLM yet.

### Deliverable

A user can select a procedure and explore:

```text
Procedure
   ↓
Diagnosis
   ↓
Clinical Concept
```

with claims-based evidence.

---

# 33. Phase 1 Timeline

### Week 1 — Data Foundation

* Load procedure data
* Load ICD-10
* Load claims
* Create canonical tables
* Convert to Parquet

### Week 2 — Relationship Construction

Build:

```text
Procedure ↔ Diagnosis
Diagnosis → Clinical Concept
Procedure ↔ Provider
Procedure ↔ POS
```

### Week 3 — Graph

* NetworkX graph
* Edge weights
* Relationship metrics
* Filtering
* Graph queries

### Week 4 — Streamlit

Build:

* Procedure Explorer
* Clinical Graph
* Relationship tables
* Basic filtering
* Methodology page

---

# 34. Phase 2 — Clinical & Utilization Intelligence

Add:

* Procedure similarity
* Provider relationships
* Specialty relationships
* POS relationships
* Temporal relationships
* Procedure sequences
* Related procedures

The question becomes:

> **"How does this procedure fit into observed healthcare utilization?"**

---

# 35. Phase 3 — Policy & Denial Intelligence

Add:

* NCD
* LCD
* Articles
* PA rules
* Denial categories
* Modifier relationships
* Coverage relationships

The graph becomes:

```text
Procedure
   ↓
Clinical Concept
   ↓
Diagnosis
   ↓
Coverage
   ↓
Authorization
   ↓
Denial
```

---

# 36. Phase 4 — ML Graph Intelligence

Add:

* Procedure similarity
* Community detection
* Relationship anomaly detection
* Provider pattern detection
* Embeddings
* Graph-based features

Potential question:

> "Which procedure/diagnosis relationships differ most from the observed population pattern?"

---

# 37. Phase 5 — AI Clinical Concept Assistant

Add conversational investigation.

Example:

> "Show me the clinical concepts associated with this procedure."

Then:

> "Which providers use it most?"

Then:

> "What other procedures are associated with those same concepts?"

Then:

> "Are there coverage policies connected to those procedures?"

The LLM becomes the natural-language interface to the graph.

---

# 38. Example End-to-End Investigation

Suppose the user selects:

```text
CPT 12345
```

The system displays:

### Procedure

```text
CPT 12345
```

### Observed diagnosis relationships

```text
Diagnosis A — 2,481 claims
Diagnosis B — 1,823 claims
Diagnosis C — 941 claims
```

### Clinical concepts

```text
Concept A
Concept B
Concept C
```

### Providers

```text
143 providers
```

### Specialties

```text
Specialty A
Specialty B
```

### POS

```text
Office
Hospital outpatient
ASC
```

### Related procedures

```text
CPT 23456
CPT 34567
CPT 45678
```

### Coverage

```text
LCD ABC
Article XYZ
```

### Denial patterns

```text
Authorization
Coding
Coverage
```

The user can then click any node to continue exploring.

---

# 39. A More Advanced Graph

Eventually your graph could look like:

```text
                         Clinical Concept
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
                Diagnosis A         Diagnosis B
                    │                   │
                    └─────────┬─────────┘
                              ↓
                         PROCEDURE
                       /      |      \
                      /       |       \
                     ↓        ↓        ↓
                Provider   Specialty   POS
                    │
                    ↓
              Related Procedure
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      Coverage               PA
          │                   │
          └─────────┬─────────┘
                    ↓
                 Denial
```

At that point, the graph becomes a reusable **healthcare intelligence layer**.

---

# 40. Relationship to Your Other Products

This product fits naturally with the other products you have been designing.

### CPT/HCPCS Procedure Intelligence

> **What is this procedure and how is it being used?**

### Procedure-to-Clinical-Concept Graph

> **What clinical concepts and healthcare activities connect to this procedure?**

### Code Change Impact Analyzer

> **What could be affected when this procedure code changes?**

### Prior Authorization Risk Engine

> **What should I investigate before the procedure occurs?**

### Denial Prevention Engine

> **What could cause the resulting claim to fail, and what can I address before submission?**

Together:

```text
                     PROCEDURE
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
       Procedure Intelligence    Code Change
             │                    Analyzer
             ↓
     Clinical Concept Graph
             │
       ┌─────┼─────┐
       ↓     ↓     ↓
     PA    Claims Denials
       │     │     │
       └─────┼─────┘
             ↓
       Denial Prevention
```

This creates a coherent **procedure-to-claim intelligence architecture**.

---

# 41. Why This Is a Strong Portfolio Project

The project demonstrates several skills simultaneously:

### Healthcare Data

* CPT/HCPCS
* ICD-10
* Claims
* NPI/provider data
* Coverage

### Data Engineering

* ETL
* Data modeling
* Parquet
* DuckDB
* Relationship construction

### Analytics

* Aggregation
* Association analysis
* Similarity
* Temporal analysis

### Graph Technology

* Nodes
* Edges
* Edge weights
* Traversal
* Network analysis

### Machine Learning

* Embeddings
* Similarity
* Clustering
* Anomaly detection

### Generative AI

* Evidence retrieval
* Graph-aware prompting
* Natural-language investigation

### Visualization

* Interactive graph
* Drill-down
* Relationship exploration
* Streamlit

That makes it substantially more technically demonstrative than simply building another claims dashboard.

---

# 42. Important Limitation

The graph represents **relationships observed in the available data**.

A frequent procedure/diagnosis pairing does not establish:

* Medical necessity
* Clinical appropriateness
* Causation
* Recommended treatment
* Clinical equivalence

Therefore, the product should consistently use language such as:

> **"observed association"**

> **"claims relationship"**

> **"utilization pattern"**

rather than making unsupported clinical conclusions.

---

# 43. Success Criteria

The MVP should allow a user to:

1. Select a CPT/HCPCS code.
2. See associated ICD-10 codes.
3. See higher-level clinical concepts.
4. See relationship strength/evidence.
5. Explore providers and specialties.
6. Explore place of service.
7. Find related procedures.
8. Filter by time period.
9. Drill into individual relationships.
10. Understand the evidence behind each relationship.

A successful demonstration should take a user from:

```text
"What is CPT 12345?"
```

to:

```text
"What clinical concepts does it connect to?"
```

to:

```text
"Who uses it?"
```

to:

```text
"What other procedures are connected?"
```

to:

```text
"What coverage, authorization, and denial relationships exist?"
```

---

# 44. Long-Term Architecture

The mature platform could become:

```text
                    DATA SOURCES
                         │
        ┌────────────────┼─────────────────┐
        ↓                ↓                 ↓
   Procedure           Claims          Clinical
     Codes                              Coding
        │                │                 │
        └────────────────┼─────────────────┘
                         ↓
                  KNOWLEDGE GRAPH
                         │
       ┌─────────────────┼──────────────────┐
       ↓                 ↓                  ↓
 Clinical            Provider           Procedure
Concepts             Network            Relationships
       │                 │                  │
       └─────────────────┼──────────────────┘
                         ↓
                  Policy Layer
                         │
                    ┌────┴────┐
                    ↓         ↓
                   PA      Coverage
                    │         │
                    └────┬────┘
                         ↓
                      Denials
                         │
                         ↓
                  ML / Anomaly Layer
                         │
                         ↓
                    LLM Assistant
                         │
                         ↓
                  Streamlit Interface
```

## Final Product Vision

The ultimate goal is not simply to draw a graph.

It is to create a **machine-readable representation of how procedure codes connect to clinical concepts and healthcare activity**, with enough evidence that analysts and AI systems can use those relationships for investigation.

The resulting platform can support the questions:

> **What is this procedure?**

> **What is it associated with?**

> **Who uses it?**

> **Where is it used?**

> **What other procedures are related?**

> **What coverage and authorization relationships exist?**

> **What denial patterns are associated with it?**

> **What changed?**

> **What should I investigate?**

That makes the Procedure-to-Clinical-Concept Graph a potential **foundational knowledge layer** underneath your entire procedure, authorization, and denial intelligence portfolio.
