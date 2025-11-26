# 🏗️ OpenHireGraph Architecture Design (Draft)

## 1. Core Philosophy: "The Graph"
The "Graph" isn't just a social graph; it's a **Semantic Knowledge Graph** connecting:
*   **Candidates** (Nodes)
*   **Skills/Concepts** (Nodes/Vectors)
*   **Experiences/Projects** (Edges/Nodes with metadata)
*   **Evidence** (Links to code, docs, portfolios)

## 2. High-Level Components

### A. Ingestion Layer (The "Senses")
Responsible for turning messy human data into structured, semantic data.
*   **Sources**: PDF Resumes, LinkedIn Exports, GitHub Profiles, Personal Websites.
*   **Process**:
    1.  **Extraction**: OCR/Text extraction.
    2.  **Normalization (LLM)**: Convert raw text into the **OpenHireGraph Schema** (JSON).
    3.  **Enrichment**: Fetch metadata for companies, verify GitHub repos.
    4.  **Embedding**: Generate vector embeddings for:
        *   The full profile summary.
        *   Specific project descriptions.
        *   Skill clusters.

### B. Storage Layer (The "Memory")
We need **Hybrid Storage** to support both semantic search and precise filtering.
*   **Primary Store (Relational + Vector)**: **PostgreSQL with `pgvector`**.
    *   *Why?* We need ACID compliance for user data AND vector search. Keeping them in one DB simplifies "Pre-filtering" (e.g., "Find vectors near 'React expert' WHERE location='London'").
*   **Blob Store**: S3/MinIO for storing original resumes/artifacts.
*   *(Optional) Graph DB*: Neo4j or Memgraph if we heavily rely on "2nd degree connections" or complex skill taxonomies, but `pgvector` might suffice for V1.

### C. Search Engine (The "Brain")
*   **Query Processor**:
    *   Input: "Senior backend engineer who knows Rust and has worked on high-throughput systems."
    *   **LLM Decomposition**: Breaks query into:
        *   *Semantic intent*: "high-throughput systems", "backend engineering"
        *   *Hard filters*: `seniority >= Senior`, `skill == Rust`
*   **Retrieval (RAG)**:
    1.  **Vector Search**: Find profiles semantically similar to the intent.
    2.  **Keyword Search**: BM25 fallback for specific niche terms.
    3.  **Re-ranking**: Cross-encoder model to score candidates based on the specific query.
*   **Synthesis**: LLM reads the top N retrieved profiles and generates a summary explaining *why* they match.

### D. API & Interface
*   **Repo Structure**: **Nx Monorepo** (pnpm workspaces).
    *   `apps/`: Backend API (Express 5), Frontend (Next.js).
    *   `libs/`: Shared schemas (Zod), database repositories, utilities.
*   **Backend**: **Node.js (TypeScript)**.
    *   *Framework*: **Express 5**.
    *   *Database Access*: **Kysely** (Type-safe SQL builder) using Repository pattern. No ORM.
    *   *AI/Vector Ecosystem*: **LangChain.js** for LLM orchestration.
*   **Frontend**: React/Next.js (TypeScript).

## 3. Data Flow

*(Diagram removed for brevity)*


## 4. Key Technical Decisions (To Discuss)
1.  **Database**: Postgres (`pgvector`) vs. Dedicated Vector DB (Qdrant/Weaviate)?
    *   *Recommendation*: Postgres for simplicity and strong relational filtering support. `pgvector` works great with Node.js (via **Kysely** for type-safe raw SQL control).
2.  **Schema Definition**: How strict should the "Open Standard" be?
    *   *Proposal*: A flexible JSON-LD based schema that allows for extension. Defined via **Zod** schemas in TypeScript for runtime validation.
3.  **Privacy**: How do we handle PII in a "Public Graph"?
    *   *Proposal*: Public profiles are anonymized/obfuscated by default until "unlocked" or are explicitly public (like GitHub).

## 5. Next Steps
1.  Define the **Candidate JSON Schema** (Zod).
2.  Set up the **Nx + Express 5 + Kysely** skeleton.
3.  Build a simple **Resume -> JSON** converter using LangChain.js.
