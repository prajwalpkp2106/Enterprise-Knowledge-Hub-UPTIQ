## Entity Breakdown (Authoritative Schema)

### **User**

```
User
-------------------------
id (PK, UUID)
email (unique)
password_hash
role (ADMIN | USER)
created_at
updated_at
```

---

### **Document**

```
Document
-------------------------
id (PK, UUID)
filename
file_path
owner_id (FK → User.id)
status (UPLOADED | PROCESSING | READY | FAILED)
created_at
updated_at
```

---

### **DocumentAccess** (Authorization Layer)

```
DocumentAccess
-------------------------
id (PK, UUID)
document_id (FK → Document.id)
user_id (FK → User.id)
access_level (READ | WRITE)
```

Purpose:

* Enables **fine-grained authorization**
* Supports future **multi-tenant & sharing**
* Required for production-grade RBAC

---

### **DocumentChunk** (AI Layer Metadata)

```
DocumentChunk
-------------------------
id (PK, UUID)
document_id (FK → Document.id)
chunk_index
content
embedding_id
```

Note:

* Stored in PostgreSQL for traceability
* Embeddings stored in vector DB (FAISS/Chroma)

---

### **QueryLog**

```
QueryLog
-------------------------
id (PK, UUID)
user_id (FK → User.id)
question
response_summary
created_at
```

Used for:

* Auditing
* Monitoring
* Analytics (Recharts in frontend)

---

## Relationships (Cardinality)

```
User 1 ────< Document
User 1 ────< DocumentAccess >──── 1 Document
Document 1 ────< DocumentChunk
User 1 ────< QueryLog
```

---

## Why This ERD Is Correct for Your Stack

### ✔ Supports Authorization

* RBAC via `User.role`
* Resource-level access via `DocumentAccess`

### ✔ Works with Microservices

* Auth Service → User
* Document Service → Document, DocumentAccess
* AI Service → DocumentChunk (read-only)

### ✔ Aligns with RAG Architecture

* Chunk traceability
* Source attribution
* Debuggable AI outputs

### ✔ Interview-Grade Design

* Clean normalization
* Extensible
* No overengineering

---

## Prisma Model Preview (Optional)

```prisma
model User {
  id       String   @id @default(uuid())
  email    String   @unique
  password String
  role     Role
  documents Document[]
}

model Document {
  id        String   @id @default(uuid())
  filename  String
  ownerId   String
  owner     User     @relation(fields: [ownerId], references: [id])
}
```
