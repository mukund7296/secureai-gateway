# secureai-gateway
SecureAI Gateway AI Security and Reliability Platform  Built a Python/FastAPI gateway for local LLM and RAG applications using Ollama and PostgreSQL. Implemented input and output guardrails, document-level access controls, source-grounded answer verification and auditable security decisions. 
# SecureAI Gateway 🛡️

**An AI security and reliability gateway for LLM and RAG applications, built with Python, FastAPI, and Ollama.**

SecureAI Gateway aims to make AI applications safer and more reliable by adding security checks, access control, source-grounded answers, and evaluation capabilities around local LLMs.

## 🎯 Problem Statement

LLM applications can generate unsupported answers, follow malicious instructions, expose sensitive information, or retrieve documents that users are not authorized to access.

SecureAI Gateway addresses these risks through a dedicated layer between the application and the LLM.

## 🚀 Key Features

The following features define the project scope. Features should be marked complete as they are implemented and tested.

- **Input Guardrails:** Validate user requests and detect common prompt-injection patterns.
- **PII Protection:** Detect and redact supported patterns of personally identifiable information.
- **Access Control:** Apply authentication, role-based authorization, and document-level permissions.
- **Secure RAG:** Retrieve authorized document chunks and return answers with source citations.
- **Hallucination Mitigation:** Detect insufficient evidence and abstain when an answer cannot be adequately supported.
- **Output Validation:** Check generated responses against configurable policies.
- **Security Audit Trail:** Record policy decisions and security events without unnecessarily storing sensitive content.
- **Evaluation Framework:** Measure answer quality, citation support, security behavior, and latency.
- **Automated Testing:** Test guardrails, authorization, retrieval, API behavior, and failure scenarios.

> **Security note:** No single guardrail can eliminate hallucinations or prompt injection. The system combines deterministic controls, access checks, source verification, and testing to reduce risk.

## 🏗️ Architecture

```text
                 Client / API Consumer
                          |
                          v
                   FastAPI Gateway
                          |
             +------------+------------+
             |            |            |
             v            v            v
        Authentication  Input       Policy
        & Authorization Guardrails  Checks
             |            |            |
             +------------+------------+
                          |
                          v
                 Authorized Retrieval
                          |
                          v
                    Ollama LLM
                          |
                          v
                 Output Verification
                          |
                 +--------+--------+
                 |                 |
                 v                 v
            Supported Answer   Abstain / Reject
                 |
                 v
            Audit & Metrics
```

### Technology Stack

| Technology | Purpose |
|---|---|
| Python | Application development |
| FastAPI | REST API |
| Pydantic | Input and output validation |
| Ollama | Local LLM inference |
| PostgreSQL | Application data and audit events |
| pgvector | Vector-based document retrieval |
| pytest | Automated testing |
| HTTPX | API integration testing |
| Docker Compose | Local environment orchestration |
| OpenTelemetry | Request tracing and observability |

## 📁 Project Structure

```text
secureai-gateway/
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── logging.py
│   ├── api/
│   │   ├── auth.py
│   │   ├── chat.py
│   │   ├── documents.py
│   │   ├── policies.py
│   │   └── evaluations.py
│   ├── schemas/
│   ├── services/
│   │   ├── ollama_service.py
│   │   ├── retrieval_service.py
│   │   ├── guardrail_service.py
│   │   ├── answer_verifier.py
│   │   └── audit_service.py
│   ├── repositories/
│   └── db/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── security/
│   └── evaluation/
├── data/
│   └── sample_docs/
├── scripts/
├── migrations/
├── docker-compose.yml
├── Dockerfile
├── .env.example
├── pyproject.toml
└── README.md
```

## ⚙️ Getting Started

### Prerequisites

- Python 3.11 or newer
- [Ollama](https://ollama.com/)
- Git
- PostgreSQL with pgvector for the database-backed RAG implementation

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/secureai-gateway.git
cd secureai-gateway
```

Replace `YOUR_USERNAME` with your GitHub username.

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows:

```powershell
.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

Install the project dependencies once they are defined in `pyproject.toml`. For an initial prototype, the basic packages are:

```bash
pip install fastapi "uvicorn[standard]" ollama \
  pydantic-settings pytest httpx
```

Add the database, authentication, vector-search, and observability dependencies as their components are implemented.

### 4. Start Ollama

Install Ollama, then download a supported local model:

```bash
ollama pull qwen2.5:3b
```

Start the Ollama service using your operating system's Ollama application or service manager. Verify that the model is available:

```bash
ollama list
```

The example model is a lightweight starting point. Model quality and performance will depend on your hardware and use case.

### 5. Configure environment variables

Create a local `.env` file from `.env.example`.

Example configuration:

```dotenv
APP_ENV=development
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=qwen2.5:3b
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/secureai
```

Update the database connection string for your local setup. Never commit `.env` or real credentials to GitHub.

### 6. Run the API

Once `app/main.py` defines the FastAPI application:

```bash
uvicorn app.main:app --reload
```

Open the interactive API documentation:

- API documentation: http://127.0.0.1:8000/docs
- Health endpoint: http://127.0.0.1:8000/health

These endpoints become available when they have been implemented in the application.

## 🔌 API Endpoints

Planned API contract:

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Check API health |
| GET | `/ready` | Check required dependencies |
| POST | `/auth/token` | Obtain a development access token |
| POST | `/documents` | Upload and ingest documents |
| GET | `/documents` | List authorized documents |
| POST | `/chat` | Submit a protected LLM/RAG request |
| GET | `/requests/{request_id}` | Retrieve an authorized request trace |
| GET | `/policies` | View active policies |
| POST | `/evaluations/run` | Run the evaluation suite |
| GET | `/evaluations/{run_id}` | Retrieve evaluation results |

These endpoints describe the intended interface and are not a claim that every endpoint is already implemented.

## 💬 Example Chat Request

```http
POST /chat
Content-Type: application/json
```

```json
{
  "question": "What is our refund policy?",
  "document_ids": ["refund-policy-v1"],
  "mode": "rag"
}
```

The application should authenticate the caller and derive the user identity from the validated token, not trust an identity supplied in the request body.

Example response:

```json
{
  "request_id": "req-example-001",
  "status": "answered",
  "answer": "Eligible purchases can be refunded within 30 days.",
  "citations": [
    {
      "document_id": "refund-policy-v1",
      "chunk_id": "chunk-04"
    }
  ],
  "safety": {
    "input_check": "passed",
    "output_check": "passed",
    "grounding": "supported"
  }
}
```

This is an illustrative response. In a real execution, the answer and citations must be validated against the actual source documents. If evidence is insufficient, the system should abstain rather than invent an answer.

## 🛡️ Security Design

The gateway is designed around several defense-in-depth principles:

1. **Validate inputs:** Enforce request schemas, size limits, and configurable security checks.
2. **Enforce authorization in application code:** Check permissions before accessing documents or invoking tools.
3. **Treat retrieved content as untrusted:** Documents may contain instructions intended to manipulate the model.
4. **Verify generated answers:** Check citation validity and evidence support; do not assume a fluent answer is correct.
5. **Minimize sensitive logging:** Store security decisions and trace identifiers while avoiding unnecessary exposure of raw prompts, secrets, and personal information.
6. **Fail safely:** Handle model timeouts, malformed responses, and unavailable dependencies without leaking internal details.
7. **Test continuously:** Maintain regression tests for attacks, access violations, and incorrect answers.

Prompt-injection detection based on keywords or regular expressions is only a basic signal. It must not be treated as a complete defense against malicious input.

## 🧪 Testing

Run the unit tests:

```bash
pytest tests/unit -v
```

Run the complete suite:

```bash
pytest -v
```

The test suite should cover:

- Valid and invalid input
- Common prompt-injection attempts
- PII detection and redaction
- Authentication and authorization failures
- Cross-user document-access attempts
- Citation validity and insufficient evidence
- Ollama timeouts and unavailable services
- API errors and request limits

Add an evaluation dataset containing representative questions, expected answers, source documents, and attack cases.

## 📊 Evaluation Metrics

The project aims to measure security and quality rather than relying on demonstrations alone.

| Metric | Measurement |
|---|---|
| Answer support rate | Proportion of evaluated claims supported by sources |
| Retrieval relevance | Whether relevant documents appear in retrieved results |
| Citation validity | Whether returned citations reference real, relevant chunks |
| Attack-test success rate | Proportion of defined attack attempts that bypass controls |
| False-positive rate | Proportion of legitimate requests incorrectly blocked |
| p50/p95 latency | Typical and high-percentile response times |
| Error rate | Proportion of failed requests |
| Cost per request | Estimated model and infrastructure cost where measurable |

Compare the baseline RAG application against the protected pipeline using the same evaluation dataset. Publish actual results, test conditions, and limitations; do not assume the guardrails improve every metric.

## 🗺️ Development Roadmap

### Week 1 — Core Application
- [ ] Set up FastAPI and Ollama
- [ ] Implement health checks and chat endpoint
- [ ] Build document ingestion and basic RAG
- [ ] Return source citations

### Week 2 — Security
- [ ] Add authentication and role-based access
- [ ] Implement input guardrails and PII checks
- [ ] Enforce document-level authorization
- [ ] Add output validation and audit events

### Week 3 — Evaluation
- [ ] Build a representative evaluation dataset
- [ ] Test prompt injection and access-control failures
- [ ] Measure answer grounding and citation quality
- [ ] Record latency and error metrics

### Week 4 — Hardening and Delivery
- [ ] Complete automated security and integration tests
- [ ] Add Docker Compose and CI checks
- [ ] Document architecture and threat model
- [ ] Publish reproducible benchmark results

## 🔭 Future Improvements

- Policy versioning and configurable risk levels
- Human approval for sensitive agent actions
- OpenTelemetry tracing and a monitoring dashboard
- Rate limiting, bounded retries, and model timeouts
- Multiple local and cloud model backends
- Red-team testing and continuous regression evaluation
- Multi-tenant access isolation

## ⚠️ Limitations

SecureAI Gateway is an engineering project intended to demonstrate security and reliability practices for LLM applications. Its effectiveness depends on the quality of its controls, test coverage, models, and deployment configuration.

It does not guarantee zero hallucinations, complete prompt-injection prevention, or production-grade security merely by enabling guardrails. A real deployment requires a threat model, independent security testing, careful access management, and ongoing evaluation.

## 👨‍💻 Author

**Mukund Biradar**

AI Engineer | Python | FastAPI | RAG | Agentic AI

- GitHub: https://github.com/mukund7296
- LinkedIn: https://www.linkedin.com/in/mukund-techlead

---

*Built to explore practical security, reliability, and evaluation patterns for AI applications.*
