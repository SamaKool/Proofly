# Proofly

**AI-assisted document verification and approval workflows**

Proofly is a full-stack platform for submitting documents, coordinating multi-stage reviews, and tracking verification progress. It combines a React web application, a Node.js REST API, MongoDB persistence, and a Python service for a retrieval-augmented policy assistant.

> **Project status:** This README describes the known project design. Confirm each feature, command, environment variable, and repository path against the implementation before presenting it as shipped functionality.

## Overview

Document reviews often involve several people, policy references, and approval steps. Proofly is designed to bring those activities into a single workflow:

- Submit and track document verification requests.
- Move requests through configurable or defined approval stages. *(Confirm how stages are configured in this repository.)*
- Authenticate users with JWT-based authentication.
- Ask a RAG-based assistant questions about policy content. The assistant uses embeddings and vector retrieval to find relevant context before generating a response with an LLM.

Proofly is an assistive workflow tool. AI-generated answers should be checked against the authoritative policy and should not be treated as a final verification decision unless the application explicitly implements that behavior.

## Features

The project context identifies these capabilities; verify their current implementation before claiming them as complete:

- **Verification workflow:** document submission, review, and multi-stage approval.
- **Progress visibility:** track a request as it moves through its workflow.
- **REST API:** Express-based backend for application operations.
- **JWT authentication:** authenticate requests using JSON Web Tokens.
- **Policy assistant:** retrieve relevant policy material using embeddings and vector search, then provide context-aware LLM responses.
- **Web interface:** React and Vite frontend for interacting with the platform.

Items such as file storage, file type validation, role-based access control, notifications, audit logs, and specific reviewer assignment behavior are implementation-dependent and are intentionally not asserted here.

## Architecture

The following is a logical view of the known components. Exact request routes, storage choices, and deployment topology depend on the repository configuration.

    ┌─────────────────────────────┐
    │ React + Vite web application│
    └──────────────┬──────────────┘
                   │ JSON REST requests
                   ▼
    ┌─────────────────────────────┐       ┌───────────────────┐
    │ Node.js + Express API       │──────▶│ MongoDB + Mongoose│
    │ JWT authentication          │       │ application data  │
    └──────────────┬──────────────┘       └───────────────────┘
                   │ assistant request
                   ▼
    ┌─────────────────────────────┐
    │ Python + FastAPI AI service │
    │ embeddings → vector retrieval
    │ → LLM response              │
    └─────────────────────────────┘

### Typical data flow

1. A user interacts with the React application.
2. The frontend sends requests to the Express REST API.
3. The API authenticates requests with JWTs and handles application and workflow operations.
4. Mongoose connects the API to MongoDB for the data represented in the application.
5. For policy questions, the application sends a request to the FastAPI service. The AI service retrieves relevant policy context using embeddings and vector retrieval, then uses an LLM to formulate a response.

The exact document upload path and the location of the original files are **not confirmed**. MongoDB may contain application records or file metadata, but do not infer that it stores the document files themselves. Document storage should be documented after confirming the implementation.

## Technology stack

| Area | Technology |
|---|---|
| Frontend | React, Vite |
| Backend API | Node.js, Express |
| Database / object mapping | MongoDB, Mongoose |
| Authentication | JWT |
| AI service | Python, FastAPI |
| Policy assistant | RAG, embeddings, vector retrieval, LLM |

Specific library names, versions, vector database choice, and LLM provider are configuration-specific and should be added only after checking the repository.

## Repository structure

This is an **illustrative template**, not a verified directory listing. Adjust it to match the actual project.

    proofly/
    ├── frontend/             # React + Vite application
    ├── backend/              # Node.js + Express REST API
    ├── ai-service/           # Python + FastAPI RAG service
    ├── docs/                 # Optional project and API documentation
    ├── .gitignore
    └── README.md

The repository may use different folder names or a monorepo layout. Replace these paths and the commands below with the actual package and service locations.

## Customize this README for the repository

Before publishing, compare this document with the code and update these items:

1. Replace the illustrative folder names with the real directory tree.
2. Copy the exact install and start scripts from the package manifests and Python service configuration. Confirm supported runtime versions and ports.
3. Replace the example environment variable names with the variables the services actually read. Keep secret values out of the README.
4. Add only verified API routes, authentication details, and documentation links.
5. Confirm where original documents and derived policy data are stored, and describe the actual embedding model, vector retrieval implementation, and LLM provider only if configured.
6. Mark features as implemented, in progress, or planned based on the current code. Remove any items that do not apply.
7. Add the real contribution instructions and license, or leave those sections clearly marked until decided.

## Getting started

### Prerequisites

Install the runtimes and services required by the repository:

- Node.js and npm (or the package manager specified by the frontend/backend manifests)
- Python (use the version specified by the AI service, if present)
- A MongoDB instance or connection string
- Credentials/configuration for the chosen LLM and embedding services, if required by the AI implementation

### 1. Get the code

    git clone <YOUR_REPOSITORY_URL>
    cd <YOUR_REPOSITORY_DIRECTORY>

Replace the placeholders with the repository URL and folder name. If the frontend, backend, or AI service is in a subdirectory, run each service's commands from that directory.

### 2. Configure environment variables

Create the environment files expected by the actual services. The names below are **suggested placeholders**, not confirmed variable names. Check the source code and existing example files, and never commit real secrets.

**Backend .env (illustrative):**

    PORT=<BACKEND_PORT>
    MONGODB_URI=<MONGODB_CONNECTION_STRING>
    JWT_SECRET=<LONG_RANDOM_SECRET>
    AI_SERVICE_URL=<FASTAPI_BASE_URL>

**Frontend .env (illustrative):**

    VITE_API_BASE_URL=<BACKEND_API_BASE_URL>

Vite exposes variables prefixed with VITE_ to client-side code. Do not put database credentials, JWT signing secrets, or private provider keys in frontend variables.

**AI service environment (illustrative):**

    AI_SERVICE_PORT=<AI_SERVICE_PORT>
    LLM_API_KEY=<PROVIDER_API_KEY>
    EMBEDDING_MODEL=<EMBEDDING_MODEL_NAME>
    LLM_MODEL=<LLM_MODEL_NAME>
    VECTOR_STORE_PATH_OR_URL=<VECTOR_STORE_CONFIGURATION>

The AI service may use different names or obtain model and vector store configuration another way. Include only the settings actually read by the service. Keep secrets in local environment configuration or an approved secret manager.

### 3. Install dependencies

Run commands in the corresponding service directories, adjusting paths and package managers to match the repository.

**Frontend** (example for npm):

    cd <FRONTEND_DIRECTORY>
    npm install

**Backend** (example for npm):

    cd <BACKEND_DIRECTORY>
    npm install

**AI service** (example using a Python virtual environment):

    cd <AI_SERVICE_DIRECTORY>
    python -m venv .venv

Activate the environment, then install the dependencies listed by the project (for example, from requirements.txt if one exists):

    python -m pip install -r requirements.txt

If the repository uses Poetry, uv, conda, pnpm, yarn, or another tool, use its checked-in lockfile and documented workflow instead.

### 4. Run the services

Start each service in a separate terminal. The commands below are examples only; confirm the scripts, modules, ports, and working directories in the actual repository.

**Backend API:**

    cd <BACKEND_DIRECTORY>
    npm run <BACKEND_START_SCRIPT>

**AI service:**

    cd <AI_SERVICE_DIRECTORY>
    uvicorn <FASTAPI_MODULE>:<APP_OBJECT> --reload --host 127.0.0.1 --port <AI_SERVICE_PORT>

For example, replace <FASTAPI_MODULE>:<APP_OBJECT> with the import path used by the project, such as main:app, only if that matches the code.

**Frontend:**

    cd <FRONTEND_DIRECTORY>
    npm run <FRONTEND_DEV_SCRIPT>

Set the frontend API base URL and backend AI-service URL to the addresses used in your local setup. Open the development URL printed by Vite. No default ports are assumed here because they are repository-specific.

## API documentation

API routes and schemas have not been provided, so no endpoint list is included here. Add the actual API reference when available:

- **Base URL:** <LOCAL_OR_DEPLOYED_API_BASE_URL>
- **API reference or OpenAPI document:** <LINK_OR_PATH_TO_API_DOCUMENTATION>
- **Authentication format:** <CONFIRM_HEADER_AND_TOKEN_FLOW>

Document each implemented route with its method, path, authentication requirements, request fields, response shape, and common errors. If the API publishes an OpenAPI specification, link to the actual specification or interactive docs URL.

## Security and privacy

- Keep JWT signing secrets and provider API keys out of source control and client-side bundles.
- Validate and authorize requests on the server; client-side checks alone do not protect API operations.
- Apply least-privilege access to verification records and document contents.
- Confirm how uploaded files are stored, accessed, validated, retained, and deleted before handling sensitive documents.
- Use HTTPS and appropriately protected configuration outside local development.
- Treat retrieved policy passages and LLM responses as untrusted content. Show users the source context where practical and require human review for consequential decisions.
- Avoid sending sensitive document contents to external AI providers unless the data handling, consent, and provider configuration have been reviewed.
- Configure CORS, request limits, logging, and error responses according to the deployment environment.

These are security considerations for operating the project; they are not claims that each control is already implemented.

## Roadmap

Potential next steps, depending on the current implementation:

- Document the actual workflow states, approval rules, and permissions.
- Add clear source references to policy assistant responses.
- Document and harden document storage and access controls.
- Add operational observability and error handling across the API and AI service.
- Publish verified API documentation and setup examples.
- Add automated checks and deployment guidance once those workflows exist in the repository.

## Contributing

Contributions are welcome. Before opening a pull request:

1. Check the existing issues and project conventions.
2. Keep changes focused and document any new configuration.
3. Include relevant validation steps and update this README when setup or behavior changes.

Replace this section with your preferred contribution process, code of conduct, and issue/PR links.

## License

**License: <CHOOSE_AND_CONFIRM_LICENSE>**

Add the license that applies to this repository and include its license file before describing the project as open source. Do not assume a license.

