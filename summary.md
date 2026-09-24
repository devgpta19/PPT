# Atharva Sugar ERP Chatbot

An AI-powered chatbot integrated with the Atharva Sugar ERP system. 

Employees can ask natural-language questions about ERP workflows, get step-by-step guidance, and watch module-specific training videos — all inside a single chat interface.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + Vite, Tailwind CSS, Framer Motion |
| Backend | ASP.NET Core 8, Clean Architecture |
| Database | SQL Server + Entity Framework Core 8 |
| Vector Store | Qdrant (RAG embeddings) |
| AI / LLM | Semantic Kernel (Ollama, OpenAI, Azure, Claude, Gemini, Grok, OpenRouter) |
| Auth | JWT Bearer + BCrypt password hashing |
| File Storage | Local (`wwwroot/`) · Azure Blob · AWS S3 |

---

## Architecture

```
Browser (React / Vite :5173)
       │
       ▼
ASP.NET Core 8 API (:7047)
   ├── Authentication   → JWT issue / refresh / revoke
   ├── ChatBot          → Message handling, session management, video lookup
   ├── RAG              → Document upload, chunking, embedding, retrieval
   └── Admin            → Documents, Videos, Users, Controls, Dashboard
       │
       ├── SQL Server   → Users, Sessions, Chat History, Documents, Videos
       └── Qdrant       → Vector embeddings for RAG retrieval
```

---

## ERP Modules Overview

The ERP is divided into **6 core modules**, each containing **3 submodules**. 

The chatbot surfaces:
* Context-aware guidance
* RAG-backed answers
* Module-specific training videos

---

### 1. Registration Module

Handles cane registration, farmer onboarding, and survey data entry.

| Submodule | Key Workflows |
|---|---|
| **Cane Registration** | Register cane lots, assign farmer, track status |
| **Farmer Profile** | Create / update farmer records, link land parcels |
| **Survey Entry** | Enter survey area data, map field boundaries |

*Suggested Questions:* How to register cane, Cane registration fields, Registration status report.

---

### 2. Agriculture Module

Covers crop lifecycle management from planning through fertilizer application.

| Submodule | Key Workflows |
|---|---|
| **Crop Planning** | Create crop plans, set schedules, view plan reports |
| **Field Inspection** | Schedule inspections, upload photos, generate reports |
| **Fertilizer Schedule** | Add schedules, view calendar, set application alerts |

*Suggested Questions:* Crop plan creation, Inspection checklist, Fertilizer usage report.

---

### 3. Harvesting / Transport Module

Manages the full harvest-to-mill transport chain.

| Submodule | Key Workflows |
|---|---|
| **Harvest Schedule** | Create harvest plans, view calendar, progress reports |
| **Transport Challan** | Create challans, track status, generate challan reports |
| **Weighbridge** | Record weighbridge entries, required fields, weigh reports |

*Suggested Questions:* Harvest schedule creation, Challan tracking, Weighbridge entry steps.

---

### 4. Accounts Module

Covers financial entries, payment processing, and ledger management.

| Submodule | Key Workflows |
|---|---|
| **Voucher Entry** | Create vouchers, approval workflow, voucher reports |
| **Bill Payment** | Process payments, check status, pending payments list |
| **Ledger** | View and search ledger, filter by date range, export |

*Suggested Questions:* Voucher creation, Bill payment status, Ledger export.

---

### 5. Payroll Module

Handles employee compensation, attendance, and compliance.

| Submodule | Key Workflows |
|---|---|
| **Salary Processing** | Run payroll, approval workflow, salary reports |
| **Attendance** | Mark attendance, view records, absent employee report |
| **PF & ESI** | Set up PF deductions, ESI contributions, compliance reports |

*Suggested Questions:* Salary processing run, Attendance records, PF/ESI setup.

---

### 6. Stores Module

Covers procurement, goods receipt, and inventory management.

| Submodule | Key Workflows |
|---|---|
| **Purchase Requisition** | Create PR, approval process, PR status tracking |
| **GRN** | Create GRN, verification steps, pending GRN report |
| **Inventory** | Check stock levels, stock transfer, low stock alerts |

*Suggested Questions:* PR creation, GRN verification, Inventory report.

---

### ⚡ Dynamic Department & Module Extensibility (Zero Code Required)

The architecture is **not restricted to 6 departments**. It is built on a modular data model (`ErpModule` and `ErpSubModule` database tables) allowing administrators to dynamically register and expand to any number of factory departments directly from the Admin Hub:
* **Additional Production & Auxiliary Units:** Distillery & By-Products, Co-Gen Power Plant, Quality Control Lab, Sales & Sugar Dispatch, Maintenance & Workshop, Security & Gate Pass.
* **Instant Automatic Onboarding:** When an admin uploads documents or videos for a new department, Qdrant automatically provisions the collection namespace, the left sidebar tree updates dynamically, and the AI immediately begins answering questions for that department — **without writing code or restarting services**.

---

## Core Features: RAG Pipeline

```
[Upload] → Parse → Chunk → Embed → Qdrant Vector Store
   │
[Query]  → Normalise → Embed ─────────┴─→ Retrieve → LLM → Response
```

1. Admin uploads PDF / DOCX / TXT via Admin Panel.
2. Backend extracts text and splits it into sliding-window chunks.
3. Chunks are embedded and stored securely in **Qdrant**.
4. User queries pull relevant chunks automatically to build exact LLM context.

---

## Core Features: Training Video System

* **Automated Tagging:** Videos uploaded via the Admin Panel are tagged with `module` + `submodule`.
* **File Naming:** System auto-saves files as `{module}_{submodule}_{unix-timestamp}.mp4`.
* **Dynamic Delivery:** Chatbot detects video intent from user messages and calls the video API instantly.
* **Smart UI Sync:** Playing a training video **automatically activates** the corresponding module layout in the app sidebar!

---

## System Infrastructure

### Admin Panel Capabilities
* **Documents:** Upload files per module, delete, download, re-index Qdrant.
* **Videos:** Upload MP4/WebM/MOV files per submodule.
* **Users:** Manage roles (`User` / `Admin`) and accounts.
* **Controls:** Toggle AI Chat, uploads, and view total usage stats.

### AI Provider Failover Chain
If a local model crashes, requests automatically drop down the chain:

```
Ollama (Local phi4-mini) → OpenRouter → Grok → OpenAI → Azure → Claude → Gemini
```

---

# Presentation & Technical Deep Dive

> This section contains all the presentation-ready technical details: **What we used, Why we used it, how the databases and LLMs interact, and the exact step-by-step implementation of RAG.**

---

## 1. What We Used and Why (Technology Selection Matrix)

When presenting, you will be asked why each component was chosen instead of alternatives. Use this table and rationale:

| Component | What We Used | Why We Used It (Technical & Business Rationale) |
|---|---|---|
| **Frontend Framework** | **React 18 + Vite** | Instant dev startup (<1s), blazing-fast HMR (Hot Module Replacement), component-based modular UI, efficient DOM diffing. |
| **UI Styling & Animation** | **Tailwind CSS + Framer Motion** | Utility-first CSS eliminates CSS bloat and ensures consistent design tokens. Framer Motion provides fluid slide-ins, spring animations, and smooth chat transitions. |
| **Client Storage** | **IndexedDB (`idbService.js`)** | Unlike `localStorage` (synchronous, 5MB limit, strings only), IndexedDB is asynchronous, non-blocking, and securely retains user session data, JWTs, and refresh tokens across browser restarts. |
| **Backend Framework** | **ASP.NET Core 8 (Web API)** | Enterprise-grade performance, native dependency injection, built-in asynchronous I/O, cross-platform deployment, and robust middleware pipelines (CORS, JWT, global exception handling). |
| **Relational Database** | **SQL Server + EF Core 8** | Industry standard for enterprise ACID compliance. Stores structured relational entities: Users, Auth tokens, Chat Sessions, Message Histories, Uploaded File Metadata, and AI Usage Logs. |
| **Vector Database** | **Qdrant (Cloud / Self-hosted)** | Specialized high-performance vector search engine written in Rust. Supports high-dimensional vector cosine distance search, real-time payload filtering (scoping search by ERP module), and zero-downtime collection reindexing. |
| **Primary AI Engine** | **Microsoft Semantic Kernel + Local SLM (Phi-4 Mini via Ollama)** | Enterprise AI orchestration abstraction. Uses **Phi-4 Mini (3.8B)** locally for **100% data privacy and $0 operational cost**. Falls back to Cloud LLMs for complex reasoning. |
| **Failover Cloud LLMs** | **Grok (Groq), OpenAI (GPT-4o-mini), Azure OpenAI, Claude 3.5, Gemini 2.0 Flash, OpenRouter** | High availability guarantee. If local Ollama goes down or machine runs out of GPU/RAM, the backend automatically fails over to cloud LLMs without throwing an error to the user. |
| **Document Parsers** | **PdfPig, DocumentFormat.OpenXml, ClosedXML** | Native, pure .NET libraries to extract raw textual content from `.pdf`, `.docx`, `.xlsx`, and `.txt` files without requiring external paid SaaS document parsers or native C++ dependencies. |
| **Security & Auth** | **JWT Bearer + BCrypt Hashing** | Stateless API security with short-lived access tokens (120 mins) + sliding refresh tokens (7 days). BCrypt adds cryptographic salting with 10 work rounds to eliminate rainbow-table attacks. |

---

## 2. Why We Need Both Databases: SQL Server vs. Qdrant

A common presentation question is: *"Why do you have two databases? Why couldn't SQL Server do everything?"*

```
┌──────────────────────────────────────┐     ┌──────────────────────────────────────┐
│       SQL Server (Relational)        │     │         Qdrant (Vector DB)           │
├──────────────────────────────────────┤     ├──────────────────────────────────────┤
│ • Exact relational lookups           │     │ • Semantic meaning search            │
│ • User credentials & BCrypt hashes   │     │ • High-dimensional floating vectors  │
│ • Chat sessions & sequential history │     │ • Cosine similarity scoring          │
│ • Document metadata (paths, sizes)   │     │ • Payload filtering by ERP module    │
│ • Financial token costs & audit logs │     │ • Nearest-neighbor retrieval         │
│ • Rigid schema, foreign keys, ACID   │     │ • Unstructured knowledge chunks      │
└──────────────────────────────────────┘     └──────────────────────────────────────┘
```

### Key Differences:
1. **Keyword vs. Semantic Search:**
   - In SQL Server, running `SELECT * FROM Docs WHERE Text LIKE '%register cane%'` will **fail** if the user types *"How do I onboard a sugarcane farmer?"* because none of the keywords match.
   - In **Qdrant**, both sentences produce vectors pointing in nearly the exact same direction in mathematical space. Qdrant returns the correct document chunk immediately.
2. **Specialized Indexing (HNSW):**
   - Qdrant uses Hierarchical Navigable Small World (HNSW) graph indexing, allowing sub-10ms nearest-neighbor searches across hundreds of thousands of document chunks.
3. **Division of Responsibility:**
   - **SQL Server** guarantees transaction safety, user permissions, audit trails, and token cost tracking.
   - **Qdrant** provides lightning-fast semantic retrieval for the AI context window.

---

## 3. Vector Database (Qdrant) Deep Dive

### What is Stored in Qdrant?
In Qdrant, data is stored in **Points** inside a collection named `knowledge_base`:

```json
{
  "id": "e3b0c442-98fc-1c14-9afb-f4c596597a18",
  "vector": [0.0124, -0.0451, 0.0883, 0.0031, ..., -0.0219],
  "payload": {
    "documentId": 14,
    "fileName": "Cane_Registration_SOP.pdf",
    "chunkIndex": 2,
    "text": "Step 2: Enter the farmer's Aadhaar number and verify land parcel registration in the portal...",
    "module": "registration",
    "subModule": "cane registration"
  }
}
```

### Critical Qdrant Features Used:
* **Deterministic Point IDs:** Each chunk ID is a deterministic UUID generated by taking the **SHA-256 hash of `fileName + chunkIndex`**. If an admin re-uploads or re-indexes the document, existing vectors are updated in place (*idempotent upsert*), preventing duplicate points.
* **Vector Dimensions:** `768 dimensions` (when using `nomic-embed-text` or `gemini text-embedding-004`) or `1536 dimensions` (for OpenAI).
* **Distance Metric:** **Cosine Similarity** ($\cos(\theta) = \frac{\mathbf{A} \cdot \mathbf{B}}{\|\mathbf{A}\| \|\mathbf{B}\|}$). Measures the angle between query and document vectors (1.0 = identical meaning, 0.0 = completely unrelated).
* **Module-Scoped Filtering:** Every query can pass a metadata filter:
  `filter: { must: [ { key: "module", match: { value: "registration" } } ] }`.
  This prevents "Stores" or "Payroll" documents from leaking into cane registration answers.

---

## 4. LLM API Keys & AI Models Explained

### Why Multiple Providers & API Keys?
In an enterprise manufacturing and ERP environment (like sugar factories, mills, and agriculture yards), reliability and data governance are paramount:
1. **Offline & Zero-Cost Operations:** The primary model is **Phi-4 Mini (3.8B)** running on-premises through **Ollama**. This requires no external internet connection, incurs $0 in per-token API costs, and keeps confidential factory payroll/financial data within the local network.
2. **Failover Resilience:** If the local server undergoes heavy compute spikes, GPU memory errors, or crashes, the system automatically drops down the **7-tier Failover Chain** (`FailoverAIProvider.cs`):
   ```
   1. Ollama (phi4-mini) [Local SLM - $0]
      ↓ (fails?)
   2. OpenRouter (Llama 3.2 1B Instruct) [Free Cloud Tier]
      ↓ (fails?)
   3. Grok via Groq (Llama 3.1 8B Instant) [Fast Cloud Inference]
      ↓ (fails?)
   4. OpenAI (GPT-4o-mini) [High Accuracy Fallback]
      ↓ (fails?)
   5. Azure OpenAI [Enterprise SLA]
      ↓ (fails?)
   6. Claude 3.5 Sonnet [Complex Logic Fallback]
      ↓ (fails?)
   7. Google Gemini 2.0 Flash [High Speed Fallback]
   ```
3. **Dynamic Database Switching:** Admins can switch the primary AI provider with a single SQL query in `AIProviderSettings` without restarting the server or altering source code.
4. **Token Cost & Usage Tracking:** Every API call logs token consumption (Input Tokens, Output Tokens, Processing Time in ms, and Estimated USD Cost) into the `AiUsageLogs` table.

---

## 5. How RAG (Retrieval-Augmented Generation) is Implemented

RAG solves the two biggest problems of LLMs in business: **hallucinations** (making up wrong steps) and **stale knowledge** (LLMs do not know Atharva Sugar's custom internal ERP rules).

### Phase 1: Document Ingestion Pipeline (`RagService.cs`)

```
Admin uploads PDF / DOCX / XLSX / TXT
                │
                ▼
1. Extract Raw Text (PdfPig / OpenXml / ClosedXML)
                │
                ▼
2. Sliding Window Chunking (SlidingWindowChunkingService.cs)
   • ChunkSize    = 1500 characters
   • ChunkOverlap = 300 characters
   (Overlap prevents cutting off sentences or formulas in mid-thought)
                │
                ▼
3. Vector Embedding Generation (nomic-embed-text / Gemini)
   • Text chunk → float[768] array
                │
                ▼
4. Upsert to Qdrant Vector DB (QdrantVectorStore.cs)
   • Deterministic UUID (SHA-256 of FileName + ChunkIndex)
   • Vector + Payload { documentId, fileName, chunkIndex, text, module }
                │
                ▼
5. Save Metadata in SQL Server (KnowledgeDocuments table)
```

---

### Phase 2: Query-Time Retrieval & Answering (`ChatBotService.cs`)

When an employee asks: *"Wanted to check pending purchase requisition approvals"*:

```
User Query: "Wanted to check pending purchase requisition approvals"
                │
                ▼
Step 1: 5-Stage Query Normalization (QueryNormalizationService.cs)
   • Strips conversational clutter ("I want to", "Please tell me")
   • Normalizes ERP synonyms ("PR" → "purchase requisition", "PO" → "purchase order")
   • Canonical Output: "how to check pending purchase requisition approvals?"
                │
                ▼
Step 2: Generate Query Embedding (FailoverAIProvider.cs)
   • "how to check pending..." → float[768] vector
                │
                ▼
Step 3: Qdrant Cosine Similarity Vector Search
   • Searches collection "knowledge_base"
   • Scoped by user's active module (e.g. module = "stores")
   • Returns top 12 chunks ranked by Cosine Similarity score
                │
                ▼
Step 4: The 3 Anti-Hallucination Architectural Gates
   ┌────────────────────────────────────────────────────────────────────────┐
   │ Gate A: Out-of-Scope Gate                                             │
   │ If highest similarity score < RelevanceThreshold (e.g. 0.01 / 0.35):   │
   │ -> Hard exit! Return "Data has not been uploaded yet." DO NOT call LLM.│
   │                                                                        │
   │ Gate B: Topic-Gap Action Verb Gate                                     │
   │ Verifies that key action verbs (e.g. "approve", "cancel") appear in    │
   │ retrieved text. If entity matches but action does not, clear context   │
   │ so the system does NOT hallucinate unsupported procedures.             │
   │                                                                        │
   │ Gate C: Direct-SOP Bypass                                              │
   │ If retrieved chunks contain 3+ numbered steps ("Step 1:", "Step 2:"):  │
   │ -> Directly return the document text verbatim! ModelName = "Direct-SOP"│
   │ Guarantees 100% fidelity to official standard operating procedures!    │
   └────────────────────────────────────────────────────────────────────────┘
                │
                ▼ (If general Q&A with valid context)
Step 5: Strict Prompt Assembly & Context Injection
   • Injects strict system prompt: "Use ONLY the context below. Do NOT use 
     your own training data. If not found, say you don't know."
   • Injects past 6 conversation turns for multi-turn context memory.
   • Injects retrieved document chunks sorted by chunkIndex (document order).
                │
                ▼
Step 6: LLM Execution & Sanitization
   • Generates plain-text answer without confusing markdown symbols.
   • Saves messages to SQL Server `ChatHistories`.
   • Logs latency, tokens, and cost to `AiUsageLogs`.
```

---

## 6. Training Video System & Semantic Media Search

* **Upload & Automated Metadata Tagging:** Admin uploads an `.mp4`/`.webm` video and selects the target ERP module and submodule.
* **Storage & Serving:** Stored locally in `ChatBotApi/wwwroot/videos/` as `{module}_{submodule}_{timestamp}.mp4` and registered in the `VideoFiles` SQL table.
* **Intent Detection:**
  * When a user asks *"Show me the Cane Registration video"*, the chatbot frontend and backend recognize the video query intent.
  * Backend calls `GET /api/chatbot/v1/videos?module=registration&subModule=cane+registration` and returns rich playable video cards.
* **Smart UI Sidebar Sync:**
  * When a user plays a training video from the chat, the application automatically dispatches a state update that highlights and expands the corresponding module and submodule in the left sidebar!
* **Offline Resilience:** If backend video APIs are temporarily unreachable, frontend seamlessly falls back to a curated local `videoRegistry.js`.

---

## 7. Security, Administration & System Controls

1. **Authentication & Session Security:**
   - Standard JWT tokens with 120-minute lifetime.
   - HTTP-only refresh tokens stored in database with auto-revocation on logout.
   - User account lockout protection after repeated failed password attempts (`LoginAttemptService.cs`).
2. **Emergency Admin Controls:**
   - **AI Chat Kill-Switch:** Instantly disable AI queries across the entire mill if an issue occurs.
   - **Upload Lock:** Restrict document/video uploads during maintenance windows.
3. **Budget & Rate-Limiting Engine:**
   - **Monthly Budget Cap ($50.00 default):** If total estimated cost across all providers reaches the cap, cloud queries are automatically paused.
   - **Daily Global Limit:** 1,000 requests per day.
   - **Per-User Rate Limits:** Max 5 requests/minute, 100 requests/hour, 1,000 requests/day to prevent API abuse.

---

## 8. Presentation Guide: Slide-by-Slide Talking Points

Use this outline when presenting this project to reviewers, leadership, or evaluators:

### Slide 1: Introduction & Problem Statement
* **The Problem:** Atharva Sugar ERP has hundreds of complex business workflows spanning 6 core departments (Registration, Agriculture, Harvest/Transport, Accounts, Payroll, Stores). New employees, mill staff, and field officers struggle to navigate manual SOPs and menus.
* **The Solution:** An intelligent, context-aware AI assistant that answers questions in natural language, quotes verified company SOPs, and serves instructional videos on-demand.

### Slide 2: High-Level Architecture & Tech Stack
* **Clean Architecture:** Modular separation between API, Repositories, Domain Entities, and AI Services.
* **Frontend:** React 18 + Vite with Tailwind CSS and Framer Motion for responsive enterprise UX.
* **Dual Database Architecture:** SQL Server for transactional relational data + Qdrant for semantic vector search.

### Slide 3: The AI Strategy — Privacy, Zero Cost & Failover
* **SLM First:** Emphasize why we use **Phi-4 Mini locally via Ollama**: zero cloud API bills, offline capability for factory networks, and complete data privacy.
* **Cloud Resilience:** Explain the 7-provider failover chain. If local hardware is busy, Grok/OpenAI/Gemini pick up the request transparently.

### Slide 4: Vector Search & Qdrant
* Explain what vectors and embeddings are (converting text to mathematical representations of meaning).
* Explain why Qdrant was selected (high-speed HNSW indexing, Cosine similarity, module payload filtering).

### Slide 5: RAG Pipeline & Anti-Hallucination Architecture (Crucial Slide!)
* Walk through Ingestion: Upload → DocumentParser → 1500/300 Sliding Window Chunking → Vector Embeddings → Qdrant.
* Walk through Retrieval: Query Normalization → Embedding → Vector Search → Score Threshold Filtering.
* **Highlight the 3 Architectural Defenses:**
  1. *Out-of-Scope Gate:* No relevant document = no LLM call.
  2. *Topic-Gap Gate:* Missing action verbs prevents made-up instructions.
  3. *Direct-SOP Bypass:* Verbatim delivery of step-by-step SOPs without LLM interference.

### Slide 6: Training Video Integration & Smart UI Sync
* Demonstrate how media complements text: Employees can read the steps or watch the video in the same conversation.
* Mention the automatic synchronization between video playback and sidebar navigation.

### Slide 7: Security, Admin Capabilities & Live Demo
* Highlight JWT authentication, BCrypt hashing, and role-based access control.
* Showcase the Admin Panel: Document uploads, real-time re-indexing, AI budget caps, and rate limiting.

---

## 9. Frequently Asked Questions (Viva / Presentation Q&A Defense)

**Q1: What happens if an employee asks something completely unrelated, like "What is the capital of France?"**
> **Answer:** The query undergoes embedding and is searched against Qdrant. Because the similarity score against our ERP documents will be near zero (well below `RelevanceThreshold`), the **Anti-Hallucination Gate** triggers immediately. The system returns: *"Data for this topic has not been uploaded yet"* without ever sending the query to an LLM.

**Q2: Why did you implement Sliding Window Chunking with 300-character overlap?**
> **Answer:** If we split text rigidly at 1500 characters, a critical sentence or formula could be cut in half across two chunks. The 300-character sliding overlap ensures semantic continuity across chunk boundaries so no context is lost.

**Q3: Why do you normalize user queries before generating embeddings?**
> **Answer:** Small language models and embedding models can be phrasing-sensitive. A user asking *"Wanted to check PR"* produces a different vector than *"how to check purchase requisition"*. Our 5-step normalization pipeline converts conversational phrasing and ERP acronyms into canonical query forms, dramatically increasing Qdrant retrieval precision.

**Q4: How do you handle re-indexing when an SOP document is updated?**
> **Answer:** Each chunk is assigned a deterministic UUID generated by taking the SHA-256 hash of `fileName + chunkIndex`. When an updated version of a document is uploaded, Qdrant performs an *upsert* on matching IDs, updating the vector and text without creating duplicate points.
