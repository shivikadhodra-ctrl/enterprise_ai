# Enterprise AI Knowledge Assistant
## Integrated Mini-Project — Technical Presentation

---

### Slide 1: Title Slide
* **Title:** Enterprise AI Knowledge Assistant
* **Subtitle:** An End-to-End Containerized RAG Platform Deployed on Kubernetes with Full Observability
* **Presenter:** Ananya
* **Repository:** [https://github.com/Ananya906/enterprise-ai-assistant](https://github.com/Ananya906/enterprise-ai-assistant)
* **Key Technologies:** FastAPI, RAG (Retrieval-Augmented Generation), Docker, Kubernetes, Prometheus, Loki, Grafana, GitHub Actions

> **Speaker Notes:**
> "Good morning/afternoon everyone. Today I am presenting my Integrated Mini-Project: the Enterprise AI Knowledge Assistant. This is an end-to-end, production-grade cloud-native application that allows enterprise users to upload confidential PDF documents and query them in natural language using Retrieval-Augmented Generation, complete with Kubernetes deployment and centralized observability."

---

### Slide 2: Problem Statement & Objectives
* **The Problem:**
  - Enterprises store critical knowledge in unstructured PDFs (HR policies, manuals, compliance docs).
  - Manual searching is slow and inefficient.
  - Standard LLMs (like ChatGPT) suffer from **hallucinations** and lack access to private enterprise documents.
* **Project Objectives:**
  - Build a secure, private document ingestion and question-answering system.
  - Implement **RAG (Retrieval-Augmented Generation)** with verifiable page-level source citations.
  - Containerize the application and deploy on **Kubernetes** with persistence.
  - Implement full **observability** (metrics + centralized logging + Grafana dashboards) and an automated **CI/CD pipeline**.

> **Speaker Notes:**
> "The objective was to solve two main issues: knowledge fragmentation and LLM hallucinations. Rather than retraining or fine-tuning models, we ground the answers strictly in uploaded enterprise documents with exact page-level citations."

---

### Slide 3: System Architecture
* **High-Level Flow:**
  1. **User Tier:** Responsive Web UI & OpenAPI Swagger interface.
  2. **Application Tier (FastAPI):** Ingestion engine, text chunking, and dual-mode answering.
  3. **Storage Tier (PersistentVolumeClaim):** Persistent document storage and TF-IDF/vector index on disk (`/data`).
  4. **Observability Tier:** Prometheus scrapes `/metrics`; Promtail ships stdout logs to Loki; Grafana provides unified visualization.

```
 [Browser / Client]
         │
         ▼
 [FastAPI Backend] ───► [pypdf Ingestion] ───► [Sliding-Window Chunker]
         │                                              │
         ▼                                              ▼
 [RAG Engine] ◄──────── [Top-k Search] ◄────── [TF-IDF / Vector Store]
         │
         ├──► [LLM Generation / Extractive Fallback] ──► Answer + Citations
         │
         ├──► [/metrics] ──────► [Prometheus] ──────┐
         └──► [stdout JSON] ───► [Promtail / Loki] ──┴──► [Grafana Dashboard]
```

> **Speaker Notes:**
> "Here is our architectural pipeline. When a PDF is uploaded, it is parsed and chunked. When a question is asked, the engine computes cosine similarity over the chunks, retrieves the top matching passages, and feeds them into our answer generator with page numbers."

---

### Slide 4: RAG Pipeline & Safety Mechanisms
* **Document Ingestion:**
  - Text extracted page-by-page using `pypdf`.
  - **Sentence-Aware Chunking:** 900-character chunks with a 150-character overlap to preserve semantic context across chunk boundaries.
* **Retrieval Engine:**
  - **TF-IDF + Cosine Similarity:** Fast, deterministic, and operates **100% offline** with zero external downloads.
  - Optional support for neural dense embeddings via `FastEmbed`.
* **Zero-Hallucination & Fallback:**
  - Strict system prompt: *"Answer ONLY from the provided context. Cite sources [1], [2]. If not found, say you don't know."*
  - **Extractive Fallback:** If no external LLM API key is configured or if network fails, the system automatically uses sentence-scoring extractive answering. The app **never throws 500 errors**.

> **Speaker Notes:**
> "A key highlight of our RAG design is resilience. Even if the system has no external internet or OpenAI API key, it falls back to an extractive answering mode, ensuring continuous business availability."

---

### Slide 5: Containerization (Docker)
* **Base Image:** `python:3.11-slim` for minimal image size and attack surface.
* **Security Best Practice:**
  - Runs as a dedicated non-root user (`appuser`, UID 10001).
  - Explicit ownership of `/data` volume.
* **Built-in Health Checks:**
  - Native Docker `HEALTHCHECK` probing `http://localhost:8000/health` every 30 seconds.
* **Multi-Container Stack (`docker-compose.yml`):**
  - App, Prometheus, Loki, Promtail, and Grafana provisioned and linked via a single command.

> **Speaker Notes:**
> "For containerization, we followed enterprise security standards: running as a non-root user, minimal slim base image, integrated health checks, and a complete Docker Compose environment for seamless local testing."

---

### Slide 6: Kubernetes Deployment
* **Kubernetes Manifests:**
  - **Namespace (`k8s/00-namespace.yaml`):** Isolated `kb-assistant` namespace.
  - **ConfigMap & Secret (`k8s/10-config.yaml`):** Decoupled environment variables and optional LLM keys.
  - **PersistentVolumeClaim (`k8s/20-pvc.yaml`):** RWO volume bound to `/data` so PDFs and vector indices survive pod crashes and restarts.
  - **Deployment (`k8s/30-deployment.yaml`):**
    - `strategy.type: Recreate` (ensures single mount on RWO volume).
    - Probes: `livenessProbe` on `/health` and `readinessProbe` on `/ready`.
    - Resource management: CPU (100m–1 Core) and Memory (256Mi–1Gi).
  - **Service (`k8s/40-service.yaml`):** NodePort on port 30080.
  - **ServiceMonitor (`k8s/50-servicemonitor.yaml`):** Configures Prometheus Operator to scrape metrics automatically.

> **Speaker Notes:**
> "On Kubernetes, we deploy with a PersistentVolumeClaim so data persists across restarts. We use both liveness and readiness probes to guarantee zero-downtime health monitoring and graceful traffic routing."

---

### Slide 7: Centralized Monitoring & Logging
* **Prometheus Metrics (`/metrics`):**
  - `http_requests_total` & `http_request_duration_seconds` (HTTP throughput & latency).
  - `kb_documents` & `kb_chunks` (Knowledge base volume gauges).
  - `kb_questions_total{mode}` (Queries tracked by mode: `llm`, `extractive`, `no_context`).
  - `kb_retrieval_duration_seconds` (Vector search speed).
* **Centralized Logging (Loki + Promtail):**
  - Application logs formatted as structured single-line JSON on `stdout`.
  - Promtail ships logs to Loki; Grafana renders live log stream panels alongside performance metrics.
* **Grafana Dashboard:**
  - Pre-configured dashboard displaying KPIs, p95 latencies, question throughput, and real-time logs in one pane.

> **Speaker Notes:**
> "Observability is built into the core. Instead of standard text logging, we output structured JSON logs to stdout which Promtail aggregates into Loki. Combined with Prometheus metrics, Grafana gives operators a complete single-pane-of-glass view."

---

### Slide 8: Automated CI/CD Pipeline
* **GitHub Actions Workflow (`.github/workflows/ci-cd.yml`):**
  1. **Test Stage:** Automated linting with `flake8` and unit testing with `pytest`.
  2. **Build Stage:** Builds Docker image and publishes to GitHub Container Registry (`ghcr.io`).
  3. **K8s Smoke-Test Stage:** Automatically launches an ephemeral Kubernetes cluster using `kind`, deploys the container, and runs curl smoke tests against `/health`, `/ready`, and `/metrics`.
  4. **Deploy Stage:** Deploys new images to the target Kubernetes cluster upon merging to `main`.

> **Speaker Notes:**
> "Our CI/CD pipeline guarantees high code quality. Before any image is pushed or deployed, GitHub Actions spins up an actual Kind Kubernetes cluster in CI to verify that manifests deploy and pass health checks."

---

### Slide 9: Live Demonstration (Summary of Key Features)
* **Demonstrated Capabilities:**
  1. Uploaded a multi-page PDF document; instant chunking and indexing.
  2. Natural language question answering with page-level citations (`[1] file.pdf p.2`).
  3. Hallucination test: Out-of-context questions return *"I couldn't find anything relevant"* instead of guessing.
  4. Grafana dashboard updating in real-time with upload and query metrics.
  5. High-availability verification: Pod restart tested with 100% data preservation from PVC.

> **Speaker Notes:**
> "During our live demo, we showed how seamless the UI is, verified that citations match exact PDF pages, demonstrated zero hallucination, and watched our Prometheus and Grafana dashboards record every request in real time."

---

### Slide 10: Conclusion & Future Roadmap
* **Project Achievements:**
  - Built an end-to-end, secure, production-grade Enterprise AI Knowledge Assistant.
  - Successfully integrated RAG, Docker, Kubernetes, CI/CD, and full-stack observability.
* **Future Enhancements:**
  - **Horizontal Scaling:** Decouple vector index to an external vector database (e.g., Qdrant or Milvus) to enable multi-replica Horizontal Pod Autoscaling (HPA).
  - **OCR Integration:** Add Tesseract OCR for scanned, image-only PDFs.
  - **Enterprise Security:** Add OAuth2 / OIDC login and role-based document access control (RBAC).

---

### Evaluator Q&A Cheat Sheet (Quick Answers)
* **Q: Why RAG instead of Fine-Tuning?**
  * *A:* RAG provides real-time updates upon upload, eliminates retraining costs, and gives verifiable page citations.
* **Q: Why chunk with 150-char overlap?**
  * *A:* Overlap ensures sentences spanning boundaries aren't split in half, preserving semantic context.
* **Q: Why Recreate deployment strategy?**
  * *A:* Because the index resides on a ReadWriteOnce (RWO) volume, which cannot be attached to two pods simultaneously during rolling updates.
* **Q: Difference between `/health` and `/ready`?**
  * *A:* `/health` is for Liveness (restarts pod if dead); `/ready` is for Readiness (stops sending traffic until indexing is ready).
