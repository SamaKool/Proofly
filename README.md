# Proofly

**Document verification and multi-stage approval workflows**

Proofly is a full-stack document verification platform. Its React application supports applicant and verifier workflows, the Express API handles authentication and verification requests, and a Python/FastAPI service provides image verification capabilities. The repository also contains an experimental retrieval-augmented policy assistant.

> **Current state:** The frontend README describes the UI as in progress. Login and registration, applicant request pages, email-link handling, and verifier pages are present; the organization admin workflow builder, invitation flow, and final responsive design are planned. The policy assistant is an optional experiment and is not part of the first end-to-end verification flow.

## What it includes

- **Verification requests and approvals:** Node.js and Express APIs for organizations, verifiers, verification requests, workflow templates, workflow steps, and workflow execution.
- **Document handling:** Authenticated upload, listing, retrieval, and deletion routes. Files are currently written to the backend's local uploads/documents directory; accepted formats are PDF, JPEG, and PNG, with a 5 MB upload limit.
- **Authentication:** JWT-based authentication and role checks. The backend uses cookies for authentication.
- **Image verification service:** Python and FastAPI service with face verification and image quality routes.
- **Experimental policy assistant:** RAG endpoint that retrieves local policy sample documents from Chroma and answers using Ollama. It returns source paths with the answer; the sample policy files are illustrative, not authoritative policy.

The policy assistant has not been evaluated for retrieval quality or hallucination rate. Treat its answers as experimental and verify them against authoritative policy.

## Architecture

    ┌─────────────────────┐      JSON / HTTP       ┌──────────────────────┐
    │ React + Vite        │ ─────────────────────▶ │ Express REST API     │
    │ Frontend            │                        │ JWT / MongoDB        │
    └─────────────────────┘                        └──────────┬───────────┘
                                                              │
                              ┌───────────────────────────────┴──────────┐
                              ▼                                          ▼
                    ┌─────────────────────┐                  ┌─────────────────────┐
                    │ MongoDB + Mongoose  │                  │ FastAPI ML service  │
                    │ workflow and user   │                  │ image checks +      │
                    │ records             │                  │ optional RAG        │
                    └─────────────────────┘                  └─────────────────────┘
                              │
                              ▼
                    Backend local disk:
                    Backend/uploads/documents/

For the policy assistant, the ML service loads policy text from ML-Service/RAG/sample_docs, splits it into chunks, embeds the chunks with sentence-transformers/all-MiniLM-L6-v2, and stores them in a local Chroma index. At query time it retrieves relevant chunks and uses the Ollama llama3.2:3b model to draft an answer.

## Technology

| Area | Implementation |
|---|---|
| Frontend | React 19, Vite 8, Axios, React Router |
| Backend | Node.js, Express 5 |
| Database | MongoDB with Mongoose |
| Authentication | JSON Web Tokens, cookie parser, role middleware |
| Document uploads | Multer, local disk storage |
| ML service | Python, FastAPI, OpenCV, InsightFace, ONNX Runtime |
| Policy assistant | LangChain, Chroma, Hugging Face sentence-transformers, Ollama |

## Repository layout

    Proofly/
    ├── Backend/
    │   ├── src/
    │   │   ├── controllers/
    │   │   ├── middlewares/
    │   │   ├── models/
    │   │   ├── routes/
    │   │   └── services/
    │   ├── test/
    │   ├── .env.example
    │   ├── package.json
    │   └── server.js
    ├── Frontend/
    │   ├── src/
    │   ├── public/
    │   └── package.json
    ├── ML-Service/
    │   ├── RAG/
    │   │   ├── sample_docs/
    │   │   ├── ingest.py
    │   │   ├── qa_chain.py
    │   │   └── router.py
    │   ├── routers/
    │   ├── services/
    │   ├── tests/
    │   ├── app.py
    │   └── requirements.txt
    ├── docs/
    └── README.md

## Run locally

### Requirements

- Node.js and npm
- Python and pip
- MongoDB
- Ollama with the llama3.2:3b model, for the optional policy assistant

### 1. Clone the repository

    git clone https://github.com/SamaKool/Proofly.git
    cd Proofly

### 2. Configure and start the backend

    cd Backend
    npm ci
    copy .env.example .env

On macOS or Linux, use cp .env.example .env instead of the Windows copy command. Edit Backend/.env and set the secrets and service credentials for your environment. The checked-in example includes these variable names:

| Variable | Purpose |
|---|---|
| PORT | Backend port; the example uses 3000 |
| MONGO_URI | MongoDB connection string |
| JWT_SECRET | Secret used for JWT signing |
| FRONTEND_URL | Allowed frontend origin; the example uses http://localhost:5173 |
| ML_SERVICE_URL | ML service address; the example uses http://localhost:8000 |
| VERIFICATION_RESULT_DIGEST_KEY | Independent secret for verification result digests |
| GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN, GOOGLE_USER | Google OAuth and sender configuration used by email integrations |

Start the API from Backend:

    npm run dev

The API uses port 3000 in the example configuration. Its health check is GET http://localhost:3000/api/health.

### 3. Start the ML service

In a second terminal, from the repository root:

    cd ML-Service
    python -m venv .venv
    python -m pip install -r requirements.txt
    uvicorn app:app --reload --port 8000

Activate the virtual environment before installing or running the service. On Windows PowerShell, use .venv\Scripts\Activate.ps1; on macOS or Linux, use source .venv/bin/activate.

The ML service serves its FastAPI documentation at http://localhost:8000/docs when running.

### 4. Start the frontend

In a third terminal:

    cd Frontend
    npm ci
    npm run dev

Open http://localhost:5173. The current API client is configured to call http://localhost:3000/api; the frontend does not currently use a VITE_API_BASE_URL setting. The backend's FRONTEND_URL must allow the frontend origin.

### 5. Build the optional policy index

The RAG sample documents and Chroma index are separate from the core verification journey. From the ML-Service directory, build the local index:

    python -m RAG.ingest

Install and start Ollama, then make the configured model available:

    ollama pull llama3.2:3b

The assistant endpoint is POST http://localhost:8000/rag/query with a JSON body such as:

    {"question":"What documents are accepted for PAN verification?"}

The response contains answer and sources fields. The generated ML-Service/RAG/chroma_store index is local and is not committed to the repository.

## API reference

The backend API base path is http://localhost:3000/api. Current route groups include:

- /auth
- /email
- /organizations
- /verification-requests
- /verifier
- /workflows
- /workflow-step
- /workflow-engine

The backend also provides GET /health. See the route implementations in [Backend/src/routes](https://github.com/SamaKool/Proofly/tree/main/Backend/src/routes) for the current methods, request fields, and access requirements. The ML service provides interactive FastAPI documentation at http://localhost:8000/docs while it is running.

## Security and data notes

- Do not commit populated .env files or reuse the example secret values.
- Uploaded documents are stored on the backend's local filesystem under Backend/uploads/documents. Protect that directory and configure durable, access-controlled storage before deploying beyond local development.
- Uploads are restricted by the current Multer configuration to PDF, JPEG, and PNG files up to 5 MB.
- The API uses Helmet, rate limiting, CORS allowlisting, JWT authentication, and role middleware. Configure FRONTEND_URL and secrets for each environment.
- Do not treat the sample policy documents or experimental RAG responses as legal or compliance guidance.
- Review data retention and access controls before processing real identity documents.

## Checks

The repository defines these checks:

**Backend** (from Backend):

    npm test

This runs the backend unit and integration test scripts.

**Frontend** (from Frontend):

    npm run lint
    npm run build
    node --test src/pages/dashboard/requestStatus.test.js

**ML service** (from ML-Service):

    pytest

Run the commands from their stated directories and install the corresponding dependencies first.

## Roadmap

The Frontend README identifies the organization admin workflow builder, invitation flow, and final responsive design as planned work. Other next steps include documenting the complete API contract, evaluating the policy assistant, and defining production document storage and retention.

## Contributing

Please open an issue to discuss a larger change before submitting a pull request. Keep changes focused and include setup or validation notes when behavior changes.

