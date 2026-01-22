# Enterprise Knowledge Hub

## Full Feature, API, Service & Auth Specification

---

# 1. System Overview

**Goal**
A secure, microservices-based platform where users upload documents and query them using AI (RAG), with strict authentication, authorization, and production-ready engineering.

---

# 2. Microservices & Responsibilities

| Service          | Tech                   | Responsibility                       |
| ---------------- | ---------------------- | ------------------------------------ |
| API Gateway      | Node + Express         | Routing, auth validation, rate limit |
| Auth Service     | Node + Express         | Users, login, JWT                    |
| Document Service | Node + Express         | Docs, access control                 |
| AI Service       | Python + FastAPI       | RAG ingestion & Q&A                  |
| Frontend         | React                  | UI, auth flow                        |
| Databases        | PostgreSQL + Vector DB | Persistent storage                   |

Each service **owns its database**.

---

# 3. Authentication & Authorization (CRITICAL)

## 3.1 Authentication (JWT)

### Flow

1. User logs in
2. Auth Service validates credentials
3. JWT issued
4. Token sent to frontend
5. API Gateway validates token on every request

### JWT Payload

```json
{
  "sub": "userId",
  "email": "user@email.com",
  "role": "ADMIN | USER",
  "iat": 123456,
  "exp": 123456
}
```

---

## 3.2 Authorization (RBAC + Resource Access)

### Role-Based

| Role  | Capabilities                      |
| ----- | --------------------------------- |
| ADMIN | Upload/delete docs, manage access |
| USER  | Read docs, query AI               |

### Resource-Based

* `DocumentAccess` table enforces per-document access
* Checked in **Document Service**
* AI queries filtered by authorized document IDs

---

# 4. API Gateway (Node + Express)

## Responsibilities

* JWT verification
* Role checks
* Request forwarding
* Rate limiting
* Swagger aggregation

---

## Gateway APIs (Public)

```
POST   /auth/login
POST   /auth/register

GET    /documents
POST   /documents/upload
DELETE /documents/:id

POST   /ai/query
```

---

## Gateway Internal Functions

```ts
verifyJWT()
authorizeRole(role)
rateLimit()
proxyRequest(serviceUrl)
logRequest()
```

---

# 5. Auth Service

## Features

* User registration
* Login
* JWT issuance
* Password hashing

---

## Auth APIs

### Register

```
POST /auth/register
```

**Body**

```json
{
  "email": "string",
  "password": "string",
  "role": "ADMIN | USER"
}
```

### Login

```
POST /auth/login
```

### Get Current User

```
GET /auth/me
```

---

## Auth Service Internal Functions

```ts
hashPassword()
comparePassword()
generateJWT()
validateCredentials()
```

---

## Auth Service Database

**Tables**

* User

---

# 6. Document Service

## Features

* Upload documents
* Store metadata
* Assign access rights
* Trigger AI ingestion

---

## Document APIs

### Upload Document (ADMIN only)

```
POST /documents/upload
```

**FormData**

* file
* accessUsers[]

### Get Documents

```
GET /documents
```

### Delete Document (ADMIN only)

```
DELETE /documents/:id
```

---

## Document Service Internal Functions

```ts
validateFile()
saveFile()
createDocumentRecord()
assignDocumentAccess()
notifyAIService()
checkUserAccess()
```

---

## Authorization Logic (Example)

```ts
if (!user.isAdmin && !hasDocumentAccess(userId, docId)) {
  throw ForbiddenError;
}
```

---

## Document Service Database

**Tables**

* Document
* DocumentAccess

---

# 7. AI Service (FastAPI)

## Features

* Document ingestion
* Chunking
* Embedding
* Semantic retrieval
* Answer generation

---

## AI APIs

### Ingest Document

```
POST /ingest
```

**Body**

```json
{
  "documentId": "uuid",
  "filePath": "/path/to/file"
}
```

---

### Query

```
POST /query
```

**Body**

```json
{
  "question": "string",
  "documentIds": ["uuid"]
}
```

---

## AI Internal Functions

```python
load_document()
split_into_chunks()
generate_embeddings()
store_vectors()
retrieve_chunks()
generate_answer()
```

---

## AI Service Database

**Tables**

* DocumentChunk
* QueryLog

**Vector DB**

* FAISS / Chroma

---

# 8. Frontend (React)

## Pages

* Login
* Register
* Dashboard
* Upload Document
* Document List
* AI Q&A

---

## Frontend Functions

```ts
login()
logout()
fetchDocuments()
uploadDocument()
askQuestion()
handleTokenExpiry()
```

---

## State Management

| State    | Tool           |
| -------- | -------------- |
| API data | TanStack Query |
| Auth     | Context API    |
| Errors   | Error Boundary |

---

# 9. Validation (Zod)

### Example

```ts
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
});
```

All APIs **must** use Zod validation.

---

# 10. Logging & Monitoring

## Logging

* Winston (JSON logs)
* Request IDs
* Error stacks

## Health Checks

```
GET /health
```

---

# 11. Security Measures

| Feature          | Tool               |
| ---------------- | ------------------ |
| Headers          | Helmet             |
| CORS             | Express CORS       |
| Rate limit       | express-rate-limit |
| Password hashing | bcrypt             |
| Input validation | Zod                |

---

# 12. Data Flow Summary

```
User → React → API Gateway
    → Auth Service (JWT)
    → Document Service (Docs)
    → AI Service (RAG)
```

---

# 13. What You Will Implement (Checklist)

### Backend

* [ ] API Gateway
* [ ] Auth Service
* [ ] Document Service
* [ ] AI Service

### Frontend

* [ ] Auth flow
* [ ] Document UI
* [ ] AI chat UI

### Infra

* [ ] Dockerfiles
* [ ] docker-compose
* [ ] Env configs
