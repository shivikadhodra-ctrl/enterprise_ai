
# Enterprise AI Knowledge Assistant 🤖

An intelligent document-based question-answering system that allows users to upload PDF files and interact with their content through natural language queries. Built using Retrieval-Augmented Generation (RAG), the application retrieves relevant information from uploaded documents and generates context-aware responses with page-level source references.

The project also includes containerization, Kubernetes deployment support, automated CI/CD workflows, centralized logging, and performance monitoring.

## 🚀 Key Highlights

- **Document Management:** Upload, view, and delete PDF documents.
- **Intelligent Document Processing:** Extracts text page by page and divides it into overlapping chunks for better retrieval.
- **RAG-based Question Answering:** Retrieves relevant document sections before generating answers.
- **Flexible Retrieval:** Supports TF-IDF similarity search by default, with optional FastEmbed-based embeddings.
- **LLM Integration:** Compatible with OpenAI-style APIs, including providers such as OpenAI, Groq, Gemini-compatible services, and Ollama.
- **Fallback Responses:** Uses extractive answering when an LLM is unavailable or a generation request fails.
- **Source References:** Displays document names, page numbers, and retrieval scores alongside answers.
- **Web Interface:** Provides a browser-based interface and interactive API documentation.
- **Monitoring:** Exposes application metrics through Prometheus and provides dashboards through Grafana.
- **Centralized Logging:** Uses structured JSON logs and Loki with Promtail for log collection.

## 🏗️ System Architecture

The application follows a document-processing and retrieval workflow:

```text
                 User Interface
                       |
                       v
                  FastAPI Server
                       |
                PDF Upload / Query
                       |
                       v
                Document Processing
                       |
                Text Extraction
                       |
                 Text Chunking
                       |
                       v
                 Vector Index
                 (Persistent Storage)
                       |
                 Similarity Search
                       |
                 Relevant Context
                       |
                       v
                 LLM Generation
                       |
                 Extractive Fallback
                       |
                       v
              Answer + Page Sources
```

**Monitoring and observability:**

```text
FastAPI Application
       |
       |---- /metrics ------> Prometheus
       |
       |---- JSON Logs -----> Promtail -----> Loki
       |                                      |
       |                                      v
       |                                  Grafana
       |
       |---- Health / Ready Endpoints
```

## 🛠️ Technology Stack

| Category | Technologies |
|---|---|
| Backend | Python, FastAPI |
| Document Processing | PyPDF |
| Retrieval | TF-IDF, FastEmbed |
| AI / LLM | OpenAI-compatible APIs |
| Frontend | Web-based application |
| Containerization | Docker, Docker Compose |
| Orchestration | Kubernetes, Minikube |
| Monitoring | Prometheus, Grafana |
| Logging | Loki, Promtail |
| CI/CD | GitHub Actions |
| Testing | Pytest, Flake8 |

## ✨ Application Capabilities

### 1. PDF Processing
- Accepts PDF files up to the configured upload limit.
- Extracts text while preserving page-level information.
- Splits extracted content into overlapping chunks.
- Maintains processed documents and indexes in persistent storage.

### 2. Context-Aware Question Answering

When a user submits a question:

1. The system searches the indexed document chunks.
2. Relevant passages are selected using similarity scores.
3. Retrieved content is supplied as context to the language model.
4. The generated response is returned with supporting document references.
5. If relevant information cannot be found, the application avoids inventing an answer.

### 3. Retrieval Options

Two retrieval configurations are supported:

- **TF-IDF:** Lightweight keyword-based retrieval that works offline without downloading embedding models.
- **FastEmbed:** Optional neural embedding-based retrieval for semantic similarity.

### 4. Resilient Answer Generation

The application supports configurable LLM providers. When API credentials are missing or a generation request fails, it switches to an extractive response mechanism.

## 📡 API Reference

The backend exposes the following REST endpoints:

| Method | Endpoint | Description |
|---|---|---|
| POST | `/upload` | Upload a PDF document |
| GET | `/documents` | Retrieve the indexed document list |
| DELETE | `/documents/{id}` | Delete a document |
| POST | `/ask` | Ask questions about uploaded content |
| GET | `/health` | Check application health |
| GET | `/ready` | Check application readiness |
| GET | `/metrics` | Access Prometheus metrics |

Interactive API documentation is available at:

`http://localhost:8000/docs`

## 💻 Getting Started

### Prerequisites

Install the following tools before running the application:

- Python 3.10+
- Git
- Docker (optional)
- Docker Compose (optional)
- Minikube and kubectl (for Kubernetes deployment)
- Helm (for monitoring installation)

### 1. Clone the Repository

```bash
git clone https://github.com/shivikadhodra-ctrl/enterprise_ai.git
cd enterprise_ai
```

### 2. Create a Virtual Environment

**Windows (Git Bash):**

```bash
python -m venv .venv
source .venv/Scripts/activate
```

**Linux / macOS:**

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements-dev.txt
```

### 4. Run Tests

```bash
pytest -q
```

### 5. Start the Application

```bash
uvicorn app.main:app --reload
```

Open the application:

- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## 🔑 LLM Configuration

The application can operate without an external LLM API key using its extractive fallback mechanism.

To enable LLM-powered responses, configure these environment variables:

```bash
export LLM_API_KEY="your-api-key"
export LLM_BASE_URL="https://api.openai.com/v1"
export LLM_MODEL="gpt-4o-mini"
```

For Windows PowerShell:

```powershell
$env:LLM_API_KEY="your-api-key"
$env:LLM_BASE_URL="https://api.openai.com/v1"
$env:LLM_MODEL="gpt-4o-mini"
```

Provider-specific API URLs and model names can be configured according to the selected provider.

**Security:** Never commit API keys or other credentials to GitHub.

## 🐳 Docker Deployment

### Build the Docker Image

```bash
docker build -t enterprise-ai-assistant:latest .
```

### Run the Container

```bash
docker run -p 8000:8000 -v kbdata:/data enterprise-ai-assistant:latest
```

The application will be accessible at:

`http://localhost:8000`

## 🐳 Running the Complete Stack

Docker Compose can be used to launch the application along with its monitoring and logging services.

```bash
docker compose up --build
```

Once the services are running, access them using:

| Service | URL |
|---|---|
| Application | http://localhost:8000 |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 |

Grafana's default credentials in the supplied configuration are:

- Username: `admin`
- Password: `admin`

Change default credentials before using the stack beyond local development.

The preconfigured dashboard provides an overview of application performance, request activity, and collected logs.

## ☸️ Kubernetes Deployment

The project includes Kubernetes manifests for deploying the application with persistent document storage.

### 1. Start Minikube

```bash
minikube start --cpus 2 --memory 4096
```

### 2. Configure the Docker Environment

```bash
eval $(minikube docker-env)
```

### 3. Build the Application Image

```bash
docker build -t enterprise-ai-assistant:latest .
```

### 4. Deploy Kubernetes Resources

```bash
kubectl apply -f k8s/00-namespace.yaml
kubectl apply -f k8s/10-config.yaml
kubectl apply -f k8s/20-pvc.yaml
kubectl apply -f k8s/30-deployment.yaml
kubectl apply -f k8s/40-service.yaml
```

### 5. Verify Deployment

```bash
kubectl -n kb-assistant rollout status deployment/kb-assistant
```

### 6. Access the Application

```bash
minikube service -n kb-assistant kb-assistant --url
```

### Configure an Optional LLM Secret

```bash
kubectl -n kb-assistant create secret generic kb-assistant-secret \
  --from-literal=LLM_API_KEY=your-api-key \
  --dry-run=client -o yaml | kubectl apply -f -
```

Restart the deployment after configuring the secret:

```bash
kubectl -n kb-assistant rollout restart deployment/kb-assistant
```

## 📊 Monitoring and Centralized Logging

The project integrates monitoring and logging tools to provide visibility into application behavior.

### Monitoring with Prometheus and Grafana

Prometheus collects application metrics, while Grafana provides dashboards for visualizing them.

Install the required Helm repositories:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Create the monitoring namespace:

```bash
kubectl create namespace monitoring
```

Install Loki and Promtail:

```bash
helm install loki grafana/loki-stack \
  -n monitoring \
  --set promtail.enabled=true \
  --set grafana.enabled=false
```

Install Prometheus and Grafana:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f monitoring/kube-prometheus-values.yaml
```

Register the application metrics endpoint:

```bash
kubectl apply -f k8s/50-servicemonitor.yaml
```

Load the Grafana dashboard:

```bash
kubectl -n monitoring create configmap kb-dashboard \
  --from-file=monitoring/grafana/dashboards/kb-assistant.json

kubectl -n monitoring label configmap kb-dashboard grafana_dashboard=1
```

Access Grafana:

```bash
kubectl -n monitoring port-forward svc/monitoring-grafana 3000:80
```

To verify metrics collection, visit Prometheus and check **Status → Targets**.

To inspect application logs in Grafana Explore, use:

```logql
{app="enterprise-ai-assistant"}
```

## 🔄 CI/CD Pipeline

GitHub Actions automates the testing and deployment workflow through the configuration in `.github/workflows/ci-cd.yml`.

The pipeline contains four main stages:

| Stage | Responsibility |
|---|---|
| Test | Runs Flake8 and Pytest |
| Build | Creates a Docker image and publishes it to GitHub Container Registry |
| Kubernetes Smoke Test | Creates a temporary Kind cluster, deploys the application, and checks its endpoints |
| Deployment | Applies Kubernetes manifests to a configured cluster when the required GitHub secret is available |

The deployment stage uses the `KUBE_CONFIG` repository secret containing a Base64-encoded Kubernetes configuration.

## ⚙️ Environment Configuration

The application supports the following configuration options:

| Variable | Default | Description |
|---|---|---|
| `DATA_DIR` | `./data` | Location for uploaded PDFs and indexes |
| `EMBEDDING_BACKEND` | `tfidf` | Retrieval backend (`tfidf` or `fastembed`) |
| `CHUNK_SIZE` | `900` | Number of characters per chunk |
| `CHUNK_OVERLAP` | `150` | Overlap between consecutive chunks |
| `TOP_K` | `4` | Number of relevant chunks retrieved |
| `LLM_API_KEY` | Empty | API authentication key |
| `LLM_BASE_URL` | OpenAI endpoint | LLM provider endpoint |
| `LLM_MODEL` | `gpt-4o-mini` | Language model selection |
| `MAX_UPLOAD_MB` | `20` | Maximum PDF upload size |
| `LOG_LEVEL` | `INFO` | Application logging level |

## 📈 Application Metrics

Prometheus collects metrics related to API usage, retrieval performance, and application health.

Some of the exposed metrics include:

- `http_requests_total`
- `http_request_duration_seconds`
- `kb_documents`
- `kb_chunks`
- `kb_documents_uploaded_total`
- `kb_upload_errors_total`
- `kb_questions_total`
- `kb_retrieval_duration_seconds`
- `kb_llm_duration_seconds`
- `kb_llm_errors_total`

Default process-level CPU and memory metrics are also available.

## 📁 Project Structure

```text
enterprise-ai-assistant/
│
├── app/
│   ├── main.py
│   ├── RAG engine
│   ├── API routes
│   ├── metrics
│   ├── logging
│   └── web interface
│
├── tests/
│   └── automated tests
│
├── k8s/
│   ├── 00-namespace.yaml
│   ├── 10-config.yaml
│   ├── 20-pvc.yaml
│   ├── 30-deployment.yaml
│   ├── 40-service.yaml
│   └── 50-servicemonitor.yaml
│
├── monitoring/
│   ├── Prometheus configuration
│   ├── Promtail configuration
│   ├── Grafana dashboards
│   └── Helm values
│
├── docs/
│   ├── presentation outline
│   └── demo documentation
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml
│
├── Dockerfile
├── docker-compose.yml
├── requirements-dev.txt
└── README.md
```

*Note: The tree above is a logical overview. Keep the actual filenames and directories aligned with your repository.*

## ⚠️ Current Limitations

- The current persistent volume uses `ReadWriteOnce` storage, making the deployment suitable for a single application replica.
- Authentication and multi-user access isolation are not currently implemented.
- Scanned PDFs and image-only documents require OCR support.
- TF-IDF primarily relies on keyword similarity and may not capture deeper semantic relationships.

## 🔮 Potential Improvements

- Introduce Qdrant or PostgreSQL with pgvector for scalable vector storage.
- Add OCR support for scanned documents.
- Implement user authentication and role-based access control.
- Integrate object storage for document management.
- Introduce horizontal pod autoscaling for larger workloads.
- Improve semantic retrieval using dedicated embedding models.

## 🎯 Project Objective

This project demonstrates how document intelligence, Retrieval-Augmented Generation, backend API development, containerization, orchestration, and observability can be combined into a practical AI-powered knowledge management solution.

The goal is to make information stored in PDF documents easier to access, search, and understand through a conversational interface.

---

**Developed by Shivika Dhodra**

GitHub: [@shivikadhodra-ctrl](https://github.com/shivikadhodra-ctrl)

