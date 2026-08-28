## 📋 Project Log

### Session 1 — 2026-08-25 (00:45 – 01:03 UTC)
**Participants:** -J (Janet), Quoth (OpenClaw agent)

#### 🎬 Kickoff
-J initiated the project. She wants Quoth (her OpenClaw agent) to be able to connect to her 4Geeks Academy account using her student token, **without her having to write any code.**

**Key asks from -J:**
1. Create a `SKILL_FILE.md` to document the project
2. Document every conversation, skill, and the rationale behind each skill
3. Store her student token securely — never in skill files or commits

#### 🔐 Token Storage Decision
- Token stored in gateway config as `env.vars.FOURGEEKS_TOKEN`
- Chose `env.vars` over `secrets.defaults.env` (schema validation issues with the latter)
- Token is accessed via `process.env.FOURGEEKS_TOKEN` in exec commands
- **Never written** to any workspace file, skill doc, commit, or log

#### 🔍 Platform Discovery
Quoth explored the 4Geeks platform to find the API surface:

| Domain | Type | Status |
|---|---|---|
| `app.4geeks.com` | Main app | Protected deploy domain |
| `4geeks.com` | Marketing site | Static SPA on Vercel |
| `learn.4geeks.com` | Learning platform | Next.js SPA, login-gated |
| `breathecode.herokuapp.com` | **API Backend** | ✅ Found via `__NEXT_DATA__` config |
| `llm.4geeks.ai` | AI/LiteLLM backend | Exists, different auth required |

**Breakthrough:** API endpoint found in `__NEXT_DATA__` script tag on `learn.4geeks.com/login`
- `BREATHECODE_HOST: https://breathecode.herokuapp.com`
- Auth format: `Authorization: Token <student_token>`

#### ✅ API Test Results (Session 1)
```
GET /v1/auth/user/me  → 200 ✅
GET /v1/events/       → 200 ✅
GET /v1/certificate/  → 403 ❌ (role-restricted)
GET /v1/registry/asset → 200 ✅ (1990 assets later discovered)
```

**User discovered:**
- ID: 21291 — Name: Janet Currie — Email: jcurrie006@gmail.com
- GitHub: jcurrie006
- Role: student at **4Geeks Miami** (downtown-miami) and **4Geeks.com** (4geeks-com)
- Since: April 7, 2026

#### 📌 Decisions (Session 1)
1. ✅ Token in `env.vars.FOURGEEKS_TOKEN` — secure, never in files
2. ✅ 4 original skills proposed, build one at a time
3. ✅ API base: `https://breathecode.herokuapp.com`
4. ✅ Auth format: `Authorization: Token <token>` (capital T, not Bearer)
5. ✅ Build after planning ("build at the end")
6. ✅ Documentation in SKILL_FILE.md throughout

---

### Session 2 — 2026-08-26 (20:09 – 21:15 UTC)
**Participants:** -J (Janet), Quoth (OpenClaw agent)

#### 🎯 Skill 2 Discovery: JS Bundle Deep-Dive
-J narrowed Skill 2 specifically to **"Get My Projects"** — projects with status.

JS bundles from `learn.4geeks.com` were reverse-engineered. The app bundle at `/_next/static/chunks/pages/_app-923403caaf8a2cee.js` contained a full API client library with these endpoints:

**Key endpoint found: `GET /v1/assignment/user/me/task`**
- Returns ALL user tasks (lessons, exercises, projects, quizzes) with live statuses
- Key fields: `task_status` (PENDING/DONE), `revision_status` (PENDING/APPROVED), `task_type` (LESSON/EXERCISE/PROJECT/QUIZ)
- Also returns: `delivered_at`, `created_at`, `updated_at`, `github_url`, `live_url`, `read_at`, `reviewed_at`, `cohort`, `description` (teacher feedback)
- Full result count on first call: **206 tasks** across **23 cohorts**
- Statuses in practice: `task_status: PENDING | DONE` — `revision_status: PENDING | APPROVED`
- **Breakthrough discovery:** The `description` field contains actual teacher feedback for graded projects

**Other endpoints discovered in JS bundles:**
- `GET /v1/assignment/user/me/final_project` — final project data
- `GET /v1/assignment/academy/cohort/{id}/task` — cohort-level task view (staff/mentor role)
- `GET /v1/admissions/cohort/user` — cohort membership data
- `GET /v1/admissions/cohort/all` — all cohorts
- `POST /v1/assignment/user/me/task` — add/bulk-update tasks
- `GET /v1/feedback/user/me/survey/response` — survey/poll feedback
- `GET /v1/assignment/me/coderevision` — code review data (currently 404, requires GitHub setup)

#### 🧠 Skill 3-4 Design
-J rapidly refined the remaining skills:
- **Skill 3 (🔴 Get Pending Work):** Originally "Track" → refocused on just pending items
- **Skill 4 (📊 Get Progress Summary):** Originally "Launch" → refocused on bird's-eye progress
- All three task skills (2-3-4) share the same endpoint: `GET /v1/assignment/user/me/task` — just different filters and aggregation

#### 🧠 Skills 5-6 Design: Teacher Feedback & Revisions
When asked about graded project feedback, Quoth discovered that teacher comments are stored directly in each task's `description` field. All 27 graded projects were inspected — 26 had meaningful teacher feedback.

**Sample feedback found:**
- *Supplier Directory:* "No DELETE /suppliers/{id} endpoint — add supplier removal"
- *Milestone 1:* "I would 100% recommend you go back and fix the spaces"
- *Milestone 4:* "verifiable skill lives under repo skills/ rather than .agents/skills/"
- *Backend Architecture:* "Could not find an explicit FastAPI layout source"
- *Todo List CLI:* "assignment expects numbered save and load commands"

This confirmed Skills 5 & 6 can be built on real, available data — no extra endpoint needed.

#### 🤯 Zapier MCP & Google Integration Discovery
Using `mcporter`, Quoth discovered the server already has a **Zapier MCP** connection with live Google accounts:

**Accounts connected (all under `jcurrieopenclaw@gmail.com`):**
- ✅ Google Docs (18 actions)
- ✅ Google Sheets (36 actions)
- ✅ Gmail (22 actions)
- ✅ Google Drive (26 actions)
- ✅ Google Calendar (21 actions)

**Auto-provisioned 123 total actions** across Google apps via `auto_provision_mcp()`.

This enables the vision: 4Geeks API → Google Docs + Sheets (live tracker) → Gmail (pending reports).

**Deliverables decided:**
- 🏗️ **Phase A:** Create a Google Doc "🎓 4Geeks Project Tracker" with sections for graded, awaiting review, pending, and progress summary
- 🏗️ **Phase B:** Create a Google Sheet with structured columns (Title, Type, Status, Revision, Delivered, Cohort)
- 🏗️ **Phase C:** Gmail send — "email me my pending projects" → fresh API pull → compose → send to `jcurrie006@gmail.com`

---

## 🧠 Full Skill Architecture — All 6 Skills

### Skill 1: 🔐 Authenticate
| Field | Detail |
|---|---|
| **Purpose** | Verify token is valid and session is active |
| **Mechanism** | Call `GET /v1/auth/user/me` with stored token |
| **Inputs** | None (reads `FOURGEEKS_TOKEN` from env) |
| **Outputs** | Success/fail, user profile, roles, academies, join date |
| **API call** | `GET /v1/auth/user/me` |
| **Initiation prompts** | • "authenticate me" • "check my token" • "is my session active?" |

### Skill 2: 📋 Get My Projects
| Field | Detail |
|---|---|
| **Purpose** | Retrieve all assigned projects with status (pending/submitted/graded) |
| **Mechanism** | Call endpoint, filter `task_type=PROJECT`, map status |
| **Inputs** | None (token from env); optional: task_type filter |
| **Outputs** | Total count, breakdown by status, lists grouped by status, delivery dates |
| **Status mapping** | `PENDING` → 📋 Pending / `DONE + revision PENDING` → 👀 Awaiting Review / `DONE + revision APPROVED` → ⭐ Graded |
| **API call** | `GET /v1/assignment/user/me/task` (filter: `task_type=PROJECT`) |
| **Initiation prompts** | • "get my projects" • "show me all my projects" • "what projects do I have?" • "project status" |

### Skill 3: 🔴 Get Pending Work
| Field | Detail |
|---|---|
| **Purpose** | Show only the work that still needs to be done |
| **Mechanism** | Call endpoint, filter `task_type=PROJECT` AND `task_status=PENDING` |
| **Inputs** | None (token from env); optional task_type (default PROJECT) |
| **Outputs** | Clean list of pending items with titles and cohort context |
| **API call** | `GET /v1/assignment/user/me/task` (filter: `task_type=PROJECT` + `task_status=PENDING`) |
| **Initiation prompts** | • "what's pending?" • "show me my pending projects" • "what do I still need to do?" • "what's on my to-do list?" |

### Skill 4: 📊 Get Progress Summary
| Field | Detail |
|---|---|
| **Purpose** | High-level overview of how far along in the course |
| **Mechanism** | Aggregate all tasks into dashboard: overall %, by-type breakdown, cohort count |
| **Inputs** | None (token from env) |
| **Outputs** | Overall %, total tasks, by-type breakdown (LESSON/EXERCISE/PROJECT), cohort count, time span, visual progress bars |
| **API call** | `GET /v1/assignment/user/me/task` (agnostic aggregate) |
| **Initiation prompts** | • "how am I doing?" • "show me my progress" • "what's my completion percentage?" • "progress summary" • "how far along am I?" |

### Skill 5: 📝 Read Teacher Feedback
| Field | Detail |
|---|---|
| **Purpose** | For graded projects, read teacher feedback, interpret it in plain English, and identify action items |
| **Mechanism** | Call endpoint, filter `task_type=PROJECT` + `revision_status=APPROVED`, extract `description` field (contains teacher feedback) |
| **Inputs** | None (token from env); optional project name filter |
| **Outputs** | For each graded project: teacher's feedback, plain-English translation, specific changes requested |
| **API call** | `GET /v1/assignment/user/me/task` (filter: `task_type=PROJECT` + `revision_status=APPROVED`) |
| **Note** | The `description` field is reused to store teacher comments. This is confirmed working — 26 of 27 graded projects have real feedback here |
| **Initiation prompts** | • "what did my teacher say?" • "show me feedback on my graded projects" • "read my teacher's feedback" • "what changes do I need to make?" • "feedback on [project name]" |

### Skill 6: 🔧 Help Complete Revisions
| Field | Detail |
|---|---|
| **Purpose** | After interpreting feedback, help implement the requested changes — open repos, write code, test |
| **Mechanism** | Uses `github_url` from task to locate repo/branch, reviews current code, implements fixes, runs tests |
| **Inputs** | Project name (which project to revise); feedback from Skill 5 |
| **Outputs** | Code changes implemented, tested, and ready to re-submit |
| **Dependencies** | Relies on Skill 5 for the list of what needs changing; uses `github_url` from task data |
| **Initiation prompts** | • "help me fix [project name]" • "help me complete the revisions" • "let's fix what the teacher flagged" • "help me redo [project name]" |

---

## 🛠️ Skills & Tools — Quick Reference

| # | Skill | Emoji | Endpoint | Filter | Status |
|---|---|---|---|---|---|
| 1 | Authenticate | 🔐 | `GET /v1/auth/user/me` | — | 🏗️ Designed |
| 2 | Get My Projects | 📋 | `GET /v1/assignment/user/me/task` | `task_type=PROJECT` | ✅ Designed |
| 3 | Get Pending Work | 🔴 | `GET /v1/assignment/user/me/task` | `task_type=PROJECT` + `task_status=PENDING` | ✅ Designed |
| 4 | Get Progress Summary | 📊 | `GET /v1/assignment/user/me/task` | Aggregated (all types) | ✅ Designed |
| 5 | Read Teacher Feedback | 📝 | `GET /v1/assignment/user/me/task` | `task_type=PROJECT` + `revision_status=APPROVED` | ✅ Designed |
| 6 | Help Complete Revisions | 🔧 | Uses `github_url` from tasks | Per-project | 🏗️ Designed |

---

## 🗺️ Google Integration Plan

**All accounts connected** via Zapier MCP under `jcurrieopenclaw@gmail.com`

| Service | Actions | Purpose |
|---|---|---|
| Google Docs | 18 (create, append, insert, find, read) | Formatted project tracker with sections |
| Google Sheets | 36 (create, add/update rows, find) | Structured data — sortable, filterable |
| Gmail | 22 (send, search, draft) | Email pending project reports |
| Google Drive | 26 (file management) | Storage and file operations |

**Phase A — Google Doc:** Create "🎓 4Geeks Project Tracker" with sections for Graded, Awaiting Review, Pending, and Progress Summary. Updated on-demand via "sync my tracker."

**Phase B — Google Sheet:** Same data as structured rows: Title, Type (PROJECT/EXERCISE), Status (PENDING/DONE), Revision (PENDING/APPROVED), Delivered (date), Cohort, GitHub URL.

**Phase C — Gmail Send:** When prompted ("email me my pending projects"), pull fresh from 4Geeks API → compose email → send to `jcurrie006@gmail.com` with clean list of pending items.

---

## 🔐 Secrets & Security

| Key | Type | Storage | Notes |
|---|---|---|---|
| `FOURGEEKS_TOKEN` | Student API Token | `env.vars` in gateway config | Never in files/repos, never logged |

---

## 📝 Notes & Gotchas

- The API `breathecode.herokuapp.com` is a Django REST backend (Heroku-era naming)
- Token auth format is `Token` (capital T), not `Bearer`
- Some endpoints require specific role permissions (403 = role-gated)
- The LLM backend at `llm.4geeks.ai` uses LiteLLM for Rigobot AI mentor
- **Critical discovery:** `description` field on tasks doubles as teacher feedback storage — confirmed for 26/27 graded projects
- Skills 2-6 all use `GET /v1/assignment/user/me/task` — just different filters/aggregation
- Code revision endpoint (`GET /v1/assignment/me/coderevision`) returns 404 — requires GitHub repo setup with user as committer
- Zapier MCP auto-provisioned 123 actions across 5 Google apps in Session 2
- Email to be sent from `jcurrieopenclaw@gmail.com` (connected) to `jcurrie006@gmail.com` (personal)
