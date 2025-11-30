# AI-Generated Documentation from Codebases – V0 Spec

## Vision

Upload a repo → get a structured docs view of the codebase.

V0 goal: **From repo → parsed structure → UI**, without any LLMs.

---

## V0 User Story

- As a user, I can:
  1. Upload a small repository as a `.zip` file.
  2. Wait while the backend processes it.
  3. View a project page that shows:
     - A list of files on the left.
     - For a selected file, a list of top-level functions and classes.
     - A basic auto-generated description like:  
       `Function <name> in <file>`.

---

## Functional Requirements

### 1. Upload & project creation

- User uploads a `.zip` file via a web UI.
- Backend:
  - Accepts the file.
  - Creates a `Project` record with status `processing`.
  - Unzips the repo into a project-specific folder.
  - Triggers the parsing pipeline.

### 2. Parsing / Core engine

Given a project’s repo folder path:

- Walk the directory tree.
- Filter to `.ts`, `.tsx`, `.js`, `.jsx`.
- For each file, extract:
  - `filePath`
  - `functions[]` (names, maybe signatures)
  - `classes[]` (names, maybe signatures)
- Build a `ProjectIR` structure:
  - `files[]` with their extracted entities.

No LLM usage in V0 – only static analysis via AST.

### 3. Persistence (Storage layer)

We need to persist:

- **Project**
  - `id` (string)
  - `name` (string)
  - `status` (`"processing" | "ready" | "failed"`)
  - `createdAt`, `updatedAt`

- **FileDoc**
  - `id`
  - `projectId` (FK → Project)
  - `filePath` (string)
  - `entities` (JSON)
    - Contains functions/classes and simple descriptions.

For V0, use **SQLite + Prisma**.

### 4. API / Backend layer

Expose the following conceptual endpoints:

- `POST /api/upload`
  - Request: `multipart/form-data` with `.zip` file.
  - Behaviour:
    - Creates `Project` with `processing`.
    - Saves + unzips the file.
    - Calls parsing engine.
    - Updates DB with `FileDoc` records.
    - Sets project `status = "ready"` on success.

- `GET /api/project/:id`
  - Returns project metadata (id, name, status, counts).

- `GET /api/project/:id/files`
  - Returns list of files (id, filePath) for the project.

- `GET /api/project/:id/files/:fileId`
  - Returns details for one file:
    - `filePath`
    - `entities` (functions, classes).

### 5. Frontend (Web UI)

**Routes:**

- `/`
  - Landing page.
  - Upload form for `.zip`.
  - List of existing projects with status.

- `/project/:id`
  - Shows:
    - Left sidebar: list of files for this project.
    - Main area: selected file details.
  - If `status = processing`, show loading state.
  - If `status = failed`, show error state.

**Responsibilities:**

- Call backend APIs.
- Display:
  - List of projects.
  - Project overview (status, counts).
  - File list + file details.
- Handle loading & error states gracefully.

---

## Tech Stack (V0)

- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Database:** SQLite + Prisma
- **Code parsing:** `ts-morph` for TS/JS AST
- **File handling:** Node.js `fs` + a zip library (`adm-zip` or similar)
- **Runtime:** Node.js (local dev; future: AWS)

---

## Non-Goals for V0

- No LLM / AI generation.
- No authentication.
- No support for languages other than TS/JS.
- No scalability/queueing (single-process, dev-focused app).
