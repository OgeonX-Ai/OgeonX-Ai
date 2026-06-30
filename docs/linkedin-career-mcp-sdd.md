# Software Design Document: LinkedIn Career Automation MCP

**Owner:** Kim Harjamäki / OgeonX AI  
**Date:** 2026-06-30  
**Status:** Draft v0.1 for Codex App implementation  
**Target repo name:** `linkedin-career-mcp` or `OgeonX-Ai/linkedin-career-mcp`  
**LinkedIn app:** `Kim Career Automation`  
**LinkedIn Client ID:** `77ovquwfa9esh0`  
**LinkedIn page:** `https://www.linkedin.com/company/135254511/`  

> Never commit the LinkedIn Client Secret, access tokens, refresh tokens, browser session cookies, CV private variants, or `.env` files.

---

## 1. Executive summary

Build a remote MCP server that connects ChatGPT / Codex / compatible MCP clients to a personal career automation backend. The system must maximize what is possible with approved LinkedIn products while staying secure, auditable, and reproducible.

The first version focuses on:

1. LinkedIn OAuth using approved scopes.
2. Authenticated LinkedIn profile read via OpenID Connect.
3. Safe LinkedIn posting/share workflows using `w_member_social`.
4. Job-hunt CRM: roles, companies, contacts, applications, follow-ups, statuses, and generated application answers.
5. CV/profile tailoring assistant backed by Kim's default role profile.
6. Outreach and recruiter-message drafting.
7. Optional browser-assist lane for user-controlled web workflows, kept separate from the official LinkedIn API lane.

The system must not claim to have LinkedIn Easy Apply API access unless LinkedIn grants an official product or partner permission. Official LinkedIn products available to this app currently do not provide a normal job-seeker Easy Apply API.

---

## 2. Goals

### 2.1 Product goals

- Create a working MCP server that can be entered into the ChatGPT Custom App screen as a trusted server URL.
- Use LinkedIn OAuth correctly and securely.
- Give ChatGPT tools for career automation:
  - read authenticated LinkedIn user info,
  - create draft LinkedIn posts,
  - publish LinkedIn posts only after explicit approval,
  - track job leads and application status,
  - generate ATS-friendly answers,
  - generate role-specific CV/cover-letter text,
  - generate recruiter outreach,
  - create a daily job-hunt dashboard.
- Design the system so Codex can build it incrementally from this SDD.

### 2.2 Engineering goals

- Production-grade but small MVP.
- Local-first development on Kim's Windows/WSL setup.
- Deployable remote HTTPS MCP endpoint.
- Secure-by-default: fail closed, least privilege, token isolation, no secret leakage.
- ISO 27000 style controls: access control, audit logging, change control, incident handling, backup/restore.
- Scrum-ready backlog with clear slices.

### 2.3 Non-goals

- No LinkedIn scraping.
- No bypassing LinkedIn restrictions.
- No storing LinkedIn password or 2FA secrets.
- No committing secrets.
- No fake job applications or false answers.
- No fully automatic public posting in MVP.
- No Easy Apply automation through unofficial API claims.

---

## 3. Current external constraints

### 3.1 LinkedIn products currently usable

The app currently has:

- `Share on LinkedIn`
- `Sign in with LinkedIn using OpenID Connect` if approved and visible in Auth scopes

Expected scopes:

```text
openid
profile
email
w_member_social
```

Actual scopes must always be read from LinkedIn Developer Portal:

```text
LinkedIn Developer Portal → Kim Career Automation → Auth → OAuth 2.0 scopes
```

### 3.2 What LinkedIn allows through current app products

- OpenID Connect: identify/authenticate the member, read lite profile claims and userinfo.
- Share on LinkedIn: create LinkedIn posts/shares on behalf of the authenticated member.

### 3.3 What is not available by default

- Easy Apply API.
- Jobs search API for personal job hunting.
- Applicant submission API for job seekers.
- Recruiter/ATS APIs without partner approval.
- Apply with LinkedIn for new partners.

---

## 4. High-level architecture

```text
ChatGPT / Codex / MCP Client
        |
        | HTTPS MCP transport
        v
Remote MCP Server: linkedin-career-mcp
        |
        | Internal service calls
        v
Career Automation Core
        |
        +--> LinkedIn OAuth Broker
        |       +--> LinkedIn OAuth endpoints
        |       +--> LinkedIn userinfo API
        |       +--> LinkedIn UGC post API
        |
        +--> Career CRM Database
        |       +--> jobs
        |       +--> companies
        |       +--> contacts
        |       +--> applications
        |       +--> follow_ups
        |       +--> generated_answers
        |       +--> audit_events
        |
        +--> CV/Profile Engine
        |       +--> Kim's profile facts
        |       +--> role fit scoring
        |       +--> ATS answer generation
        |
        +--> Posting Workflow
        |       +--> draft
        |       +--> review
        |       +--> publish after explicit approval
        |
        +--> Optional Browser Assist Worker
                +--> local-only Playwright/browser tasks
                +--> separated from LinkedIn API tokens
                +--> consent-gated actions
```

---

## 5. Recommended stack

### 5.1 Server

Use Python for fastest MVP and strong Azure alignment:

```text
Python 3.12+
FastAPI
FastMCP or MCP Python SDK
httpx
pydantic v2
SQLAlchemy or SQLModel
SQLite for local dev
PostgreSQL for production
pytest
ruff
mypy optional
```

Alternative if Codex prefers TypeScript:

```text
Node.js 22+
TypeScript
Express/Fastify
MCP TypeScript SDK
Prisma
PostgreSQL
Vitest
ESLint
```

Primary recommendation: **Python FastAPI + FastMCP**.

### 5.2 Deployment

MVP deployment options:

1. Local dev with tunnel:
   - `uvicorn` locally
   - dev tunnel / ngrok / Cloudflare Tunnel
2. Azure Container Apps:
   - containerized FastAPI MCP server
   - managed identity
   - Azure Key Vault
   - Azure PostgreSQL or SQLite only for private MVP
3. Azure Functions is possible, but less ideal for long-lived MCP transports.

Primary deployment target:

```text
Azure Container Apps + Key Vault + PostgreSQL
```

---

## 6. Repository structure

```text
linkedin-career-mcp/
  README.md
  AGENTS.md
  .gitignore
  .env.example
  pyproject.toml
  ruff.toml
  Dockerfile
  docker-compose.yml
  docs/
    linkedin-career-mcp-sdd.md
    threat-model.md
    oauth-flow.md
    mcp-tools.md
    deployment.md
  src/
    linkedin_career_mcp/
      __init__.py
      main.py
      settings.py
      logging_config.py
      mcp_server.py
      auth/
        __init__.py
        linkedin_oauth.py
        token_store.py
        state_store.py
        crypto.py
      linkedin/
        __init__.py
        client.py
        models.py
        posts.py
        profile.py
      career/
        __init__.py
        profile_facts.py
        role_fit.py
        answer_generator.py
        application_tracker.py
        outreach.py
      db/
        __init__.py
        models.py
        session.py
        migrations/
      tools/
        __init__.py
        tool_profile.py
        tool_posts.py
        tool_jobs.py
        tool_answers.py
        tool_outreach.py
      web/
        __init__.py
        health.py
        oauth_routes.py
        mcp_routes.py
  tests/
    test_health.py
    test_linkedin_oauth.py
    test_tool_schemas.py
    test_career_answers.py
    test_security_no_secrets.py
  scripts/
    dev.ps1
    dev.sh
    smoke_test.ps1
    rotate_secret_check.py
  .github/
    workflows/
      ci.yml
```

---

## 7. Runtime components

### 7.1 MCP server layer

Responsible for:

- Exposing MCP tools.
- Advertising server capabilities.
- Handling MCP transport.
- Returning structured JSON outputs.
- Mapping user requests to safe internal services.

Initial tools:

```text
linkedin.get_user_info
linkedin.create_post_draft
linkedin.publish_post_after_approval
career.add_job_lead
career.list_job_leads
career.update_application_status
career.generate_application_answers
career.generate_cover_note
career.generate_recruiter_outreach
career.daily_job_hunt_brief
security.get_audit_log
```

### 7.2 OAuth broker

Responsible for:

- Redirecting user to LinkedIn authorization.
- Validating `state`.
- Exchanging authorization code for access token.
- Storing tokens encrypted.
- Refresh/re-auth logic when tokens expire.
- Never logging secrets or tokens.

The OAuth broker must support:

```text
GET /auth/linkedin/start
GET /auth/linkedin/callback
POST /auth/linkedin/disconnect
```

### 7.3 LinkedIn API client

Responsible for:

- `GET /v2/userinfo`
- `POST /v2/ugcPosts`
- later optional image upload flow if needed

Must use:

```text
Authorization: Bearer <access_token>
X-Restli-Protocol-Version: 2.0.0
```

for UGC post calls.

### 7.4 Career CRM core

Stores structured job-hunt state independent of LinkedIn restrictions.

This becomes the core value of the app:

- every role tracked,
- every answer saved,
- every follow-up scheduled,
- every company mapped,
- every gap detected,
- every recruiter conversation prepared.

### 7.5 Optional browser-assist worker

This is separate and must not be part of LinkedIn API OAuth.

Allowed uses:

- open a job page for the user,
- extract user-visible job details with user supervision,
- draft answers into local queue,
- maintain local tracker.

Blocked in MVP:

- credential handling,
- CAPTCHA bypass,
- anti-bot bypass,
- hidden scraping,
- mass automatic submit flows.

---

## 8. Data model

### 8.1 `linked_accounts`

```text
id UUID PK
provider TEXT = 'linkedin'
subject TEXT
name TEXT
email TEXT nullable
picture_url TEXT nullable
scopes TEXT
created_at TIMESTAMP
updated_at TIMESTAMP
last_authenticated_at TIMESTAMP
```

### 8.2 `oauth_tokens`

```text
id UUID PK
linked_account_id UUID FK
provider TEXT
encrypted_access_token TEXT
encrypted_refresh_token TEXT nullable
expires_at TIMESTAMP nullable
scope TEXT
created_at TIMESTAMP
updated_at TIMESTAMP
```

### 8.3 `job_leads`

```text
id UUID PK
source TEXT
source_url TEXT nullable
company_name TEXT
title TEXT
location TEXT nullable
work_mode TEXT nullable -- remote/hybrid/onsite
seniority TEXT nullable
posted_at TIMESTAMP nullable
discovered_at TIMESTAMP
status TEXT -- new, saved, applied, rejected, interview, offer, archived
fit_score INTEGER nullable
risk_flags JSON
notes TEXT
created_at TIMESTAMP
updated_at TIMESTAMP
```

### 8.4 `applications`

```text
id UUID PK
job_lead_id UUID FK
applied_at TIMESTAMP nullable
application_channel TEXT -- linkedin_easy_apply, company_site, recruiter_email, other
cv_version TEXT nullable
cover_letter_version TEXT nullable
answers JSON
status TEXT
next_action TEXT nullable
next_action_due_at TIMESTAMP nullable
created_at TIMESTAMP
updated_at TIMESTAMP
```

### 8.5 `career_profile_facts`

Seed with Kim's default facts:

```json
{
  "name": "Kim Harjamäki",
  "location": "Tampere / Helsinki, Finland",
  "target_roles": [
    "Azure Architect",
    "Cloud Architect",
    "AI Cloud Architect",
    "Platform Engineer",
    "Senior DevOps Engineer",
    "Cloud Solution Architect",
    "Integration Architect",
    "Data Platform Lead"
  ],
  "years": {
    "azure": 6,
    "devops_ci_cd": 6,
    "iac_bicep_terraform": 6,
    "security_governance": 6,
    "python": 3,
    "etl_data_pipelines": 3,
    "data_engineering": 2,
    "ai_rag_agents": 1
  },
  "languages": {
    "finnish": "Native or bilingual",
    "english": "Professional / fluent"
  },
  "work_authorization": "Authorized to work in Finland/EU",
  "sponsorship_required": false,
  "notice_period": "Immediate / negotiable",
  "remote_hybrid": true,
  "relocation": "Finland only / remote EU"
}
```

### 8.6 `post_drafts`

```text
id UUID PK
text TEXT
visibility TEXT -- PUBLIC, CONNECTIONS
article_url TEXT nullable
status TEXT -- draft, approved, published, failed
linkedin_post_id TEXT nullable
created_at TIMESTAMP
approved_at TIMESTAMP nullable
published_at TIMESTAMP nullable
```

### 8.7 `audit_events`

```text
id UUID PK
actor TEXT
action TEXT
target_type TEXT
target_id TEXT nullable
risk_level TEXT
input_hash TEXT nullable
output_hash TEXT nullable
metadata JSON
created_at TIMESTAMP
```

---

## 9. MCP tools specification

### 9.1 `linkedin.get_user_info`

Purpose:

- Validate OAuth connection.
- Retrieve LinkedIn user info through OIDC.

Input:

```json
{}
```

Output:

```json
{
  "connected": true,
  "subject": "...",
  "name": "Kim Harjamäki",
  "email": "optional",
  "picture": "optional"
}
```

Failure modes:

```text
AUTH_REQUIRED
TOKEN_EXPIRED
SCOPE_MISSING
LINKEDIN_API_ERROR
```

### 9.2 `linkedin.create_post_draft`

Purpose:

- Draft a LinkedIn post but do not publish.

Input:

```json
{
  "topic": "AI job search automation",
  "tone": "professional, direct, slightly bold",
  "target_audience": "Finnish recruiters and Azure/AI hiring managers",
  "article_url": null
}
```

Output:

```json
{
  "draft_id": "uuid",
  "text": "...",
  "status": "draft"
}
```

### 9.3 `linkedin.publish_post_after_approval`

Purpose:

- Publish a previously approved post.
- Must require explicit user approval per post.

Input:

```json
{
  "draft_id": "uuid",
  "approval_text": "Publish this LinkedIn post now"
}
```

Output:

```json
{
  "published": true,
  "linkedin_post_id": "..."
}
```

Guardrail:

- Reject if `approval_text` is missing or not explicit.

### 9.4 `career.add_job_lead`

Input:

```json
{
  "company_name": "Example Oy",
  "title": "Senior Azure Architect",
  "source_url": "https://...",
  "location": "Finland",
  "work_mode": "Hybrid",
  "description": "..."
}
```

Output:

```json
{
  "job_lead_id": "uuid",
  "fit_score": 88,
  "recommended_action": "Apply"
}
```

### 9.5 `career.generate_application_answers`

Purpose:

- Generate defensible, CV-backed answers for application forms.

Input:

```json
{
  "job_lead_id": "uuid",
  "questions": [
    "How many years of Microsoft Azure experience do you have?",
    "How many years of ETL tools experience do you have?"
  ]
}
```

Output:

```json
{
  "answers": [
    {
      "question": "How many years of Microsoft Azure experience do you have?",
      "answer": "6",
      "confidence": "high",
      "basis": "CV states 6+ years Azure Architecture."
    },
    {
      "question": "How many years of ETL tools experience do you have?",
      "answer": "3",
      "confidence": "medium",
      "basis": "CV supports Azure Data Engineering, database development, financial reporting SaaS and automation pipelines."
    }
  ]
}
```

### 9.6 `career.daily_job_hunt_brief`

Purpose:

- Give Kim a morning/evening action plan.

Output:

```json
{
  "priority_1": "Apply to 5 Azure Architect roles",
  "priority_2": "Follow up with 2 recruiters",
  "priority_3": "Publish one LinkedIn availability post",
  "blocked": ["Need current CV version uploaded to tracker"]
}
```

---

## 10. Security design

### 10.1 Secrets

Secrets must live in environment variables or Key Vault:

```text
LINKEDIN_CLIENT_ID=77ovquwfa9esh0
LINKEDIN_CLIENT_SECRET=<secret>
APP_ENCRYPTION_KEY=<32-byte key>
DATABASE_URL=<db url>
PUBLIC_BASE_URL=https://<server-domain>
```

Never log:

```text
client_secret
access_token
refresh_token
authorization code
cookies
session IDs
raw CV private data
```

### 10.2 Token storage

- Encrypt tokens at rest.
- Use envelope encryption if on Azure.
- Store token expiry.
- Redact token-like strings in logs.
- Provide disconnect/delete endpoint.

### 10.3 OAuth controls

- Validate exact redirect URI.
- Generate cryptographically random `state`.
- Validate `state` on callback.
- Use HTTPS in production.
- Do not put tokens in query strings.
- Prefer PKCE if the final MCP/ChatGPT client flow supports it.

### 10.4 Tool risk classification

| Risk | Tool | Control |
|---|---|---|
| Low | `get_user_info` | OAuth required |
| Low | `list_job_leads` | Owner-only data |
| Medium | `add_job_lead` | Audit log |
| Medium | `generate_application_answers` | CV-backed factual basis |
| High | `publish_post_after_approval` | Explicit approval text |
| High | browser-assist submit | Excluded from MVP |

### 10.5 ISO 27000-aligned controls

- Access control: one user initially, future multi-user RBAC.
- Asset management: identify token store, DB, source repo, deployment environment.
- Cryptography: encrypted token storage and HTTPS only.
- Logging and monitoring: structured audit events.
- Change management: PRs, CI checks, Codex changes reviewed.
- Incident handling: rotate secret, revoke LinkedIn app grant, clear token store.
- Supplier/API risk: LinkedIn API limits and product approval constraints documented.

---

## 11. Compliance and LinkedIn boundary

The MCP must not pretend to have products/scopes that the LinkedIn Developer Portal has not approved.

The system may:

- authenticate with LinkedIn,
- read userinfo through OIDC,
- create LinkedIn shares through approved share scope,
- manage Kim's own job-hunt tracker,
- draft application answers,
- prepare recruiter messages,
- create personal productivity workflows.

The system may not:

- access Easy Apply through an unofficial API,
- bypass LinkedIn anti-automation controls,
- scrape restricted pages,
- store LinkedIn credentials,
- submit job applications without a compliant channel and proper user authorization.

---

## 12. User experience

### 12.1 ChatGPT Custom App screen

Fill:

```text
Icon: OgeonX AI logo
Name: LinkedIn Career Automation
Description: LinkedIn OAuth, profile, posting and career automation bridge for OgeonX AI.
Connection: Server URL
Server URL: https://<deployed-server>/sse or https://<deployed-server>/mcp
Authentication: OAuth
```

### 12.2 Main user flows

#### Flow A: Connect account

```text
User clicks Connect
→ ChatGPT opens OAuth flow
→ LinkedIn consent
→ MCP server callback
→ token stored encrypted
→ userinfo tested
→ connection ready
```

#### Flow B: Draft post

```text
User asks: Draft LinkedIn post about being available for Azure AI Architect roles
→ tool creates draft
→ user reviews
→ user explicitly says publish
→ post published via LinkedIn API
```

#### Flow C: Track job lead

```text
User pastes job URL/description
→ MCP stores job lead
→ fit score calculated
→ answers generated
→ next action created
```

#### Flow D: Recruiter outreach

```text
User gives recruiter/company/role
→ MCP generates short message
→ stores contact/follow-up
→ user sends manually or later approved channel sends
```

---

## 13. API integration details

### 13.1 LinkedIn OAuth authorize

```text
GET https://www.linkedin.com/oauth/v2/authorization
```

Parameters:

```text
response_type=code
client_id=77ovquwfa9esh0
redirect_uri=https://<server-domain>/auth/linkedin/callback
state=<random>
scope=openid profile email w_member_social
```

### 13.2 LinkedIn token exchange

```text
POST https://www.linkedin.com/oauth/v2/accessToken
Content-Type: application/x-www-form-urlencoded
```

Body:

```text
grant_type=authorization_code
code=<code>
client_id=77ovquwfa9esh0
client_secret=<secret>
redirect_uri=https://<server-domain>/auth/linkedin/callback
```

### 13.3 Userinfo

```text
GET https://api.linkedin.com/v2/userinfo
Authorization: Bearer <access_token>
```

### 13.4 Text post

```text
POST https://api.linkedin.com/v2/ugcPosts
Authorization: Bearer <access_token>
X-Restli-Protocol-Version: 2.0.0
Content-Type: application/json
```

Body:

```json
{
  "author": "urn:li:person:<sub-or-person-id>",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "..."
      },
      "shareMediaCategory": "NONE"
    }
  },
  "visibility": {
    "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
  }
}
```

Note: if `userinfo.sub` does not work directly as the Person URN for UGC posts, implement a resolver or store the verified author URN from a successful test.

---

## 14. Error handling

Use normalized errors:

```text
AUTH_REQUIRED
TOKEN_EXPIRED
SCOPE_MISSING
LINKEDIN_RATE_LIMIT
LINKEDIN_API_ERROR
VALIDATION_ERROR
APPROVAL_REQUIRED
SECRET_MISSING
DB_ERROR
```

Return shape:

```json
{
  "ok": false,
  "error_code": "SCOPE_MISSING",
  "message": "The LinkedIn app does not have w_member_social scope.",
  "next_step": "Open LinkedIn Developer Portal → Products → Share on LinkedIn."
}
```

---

## 15. Observability

### 15.1 Logs

Structured JSON logs:

```json
{
  "timestamp": "2026-06-30T14:00:00Z",
  "level": "INFO",
  "event": "tool_called",
  "tool": "career.generate_application_answers",
  "user": "kim",
  "request_id": "..."
}
```

### 15.2 Metrics

- tool calls by tool name,
- OAuth success/failure,
- LinkedIn API errors,
- posts drafted,
- posts published,
- job leads added,
- applications tracked,
- follow-ups due.

### 15.3 Audit events

Every consequential action writes to `audit_events`:

- OAuth connected/disconnected,
- post draft created,
- post published,
- job lead added,
- application status changed,
- generated answer used.

---

## 16. Testing strategy

### 16.1 Unit tests

- OAuth URL generation.
- State validation.
- Token redaction.
- LinkedIn API client request construction.
- Tool input validation.
- Fit scoring.
- Default answer generation.

### 16.2 Integration tests

Use mocked LinkedIn API responses.

- `/auth/linkedin/start` returns redirect.
- `/auth/linkedin/callback` exchanges token using mocked response.
- `linkedin.get_user_info` returns normalized user info.
- `linkedin.create_post_draft` stores draft.
- `linkedin.publish_post_after_approval` requires explicit approval.

### 16.3 Security tests

- `.env` is ignored.
- No secret-like values in logs.
- Token values are encrypted at rest.
- Publishing without approval fails.
- Unknown scopes fail clearly.
- OAuth state mismatch returns 401.

### 16.4 Manual smoke test

```text
1. Run server locally.
2. Start tunnel.
3. Add tunnel callback to LinkedIn Auth redirects.
4. Connect OAuth.
5. Call get_user_info.
6. Create post draft.
7. Do not publish until explicit test approval.
8. Add one job lead.
9. Generate application answers.
```

---

## 17. Deployment plan

### 17.1 Local dev

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e .[dev]
copy .env.example .env
uvicorn linkedin_career_mcp.main:app --reload --port 8000
```

Tunnel:

```powershell
cloudflared tunnel --url http://localhost:8000
```

Set LinkedIn redirect:

```text
https://<tunnel-domain>/auth/linkedin/callback
```

### 17.2 Azure Container Apps

- Build Docker image.
- Push to Azure Container Registry or GitHub Container Registry.
- Deploy to Container Apps.
- Configure env vars from Key Vault.
- Configure custom domain + HTTPS.
- Add production callback URL to LinkedIn.

---

## 18. Scrum backlog

### Sprint 0: Repo bootstrap

Acceptance criteria:

- repo created,
- Python project initialized,
- `AGENTS.md` added,
- CI running ruff + pytest,
- `/healthz` endpoint works.

Tasks:

1. Create repo skeleton.
2. Add FastAPI app.
3. Add config handling.
4. Add CI.
5. Add Dockerfile.

### Sprint 1: LinkedIn OAuth

Acceptance criteria:

- `/auth/linkedin/start` works,
- callback validates `state`,
- token exchange implemented,
- token redaction tested,
- `get_user_info` works with real token.

Tasks:

1. Implement OAuth state store.
2. Implement LinkedIn token exchange.
3. Implement encrypted token store.
4. Implement userinfo client.
5. Add tests.

### Sprint 2: MCP tools MVP

Acceptance criteria:

- MCP server exposes tools,
- ChatGPT Custom App can connect,
- `linkedin.get_user_info` works,
- `career.add_job_lead` works,
- `career.generate_application_answers` works.

Tasks:

1. Implement MCP transport.
2. Register tools.
3. Add Pydantic schemas.
4. Add job tracker DB.
5. Add answer generator.

### Sprint 3: LinkedIn posting

Acceptance criteria:

- post draft created,
- approval required,
- publish works only after approval,
- audit event written.

Tasks:

1. Implement draft store.
2. Implement UGC post client.
3. Implement publish guardrail.
4. Add manual smoke test.

### Sprint 4: Career dashboard

Acceptance criteria:

- daily brief tool works,
- application statuses visible,
- next actions generated,
- follow-up queue works.

Tasks:

1. Implement status model.
2. Implement daily brief.
3. Implement follow-up due query.
4. Add summary output.

---

## 19. Codex App master prompt

Paste this into Codex App once the target repo exists:

```text
You are implementing the LinkedIn Career Automation MCP server for OgeonX AI.

Read docs/linkedin-career-mcp-sdd.md as the source of truth.

Build incrementally. Do not over-engineer. Do not commit secrets. Do not add LinkedIn scraping or Easy Apply claims. Use Python 3.12, FastAPI, Pydantic v2, httpx, pytest, ruff, and SQLite for local dev. Add clear .env.example placeholders.

Sprint 0 goal:
- create a working FastAPI project skeleton
- add /healthz
- add Dockerfile
- add CI workflow with ruff and pytest
- add AGENTS.md with project rules
- add tests

Guardrails:
- no client secret in source
- no access token logging
- no public posting tool yet
- no browser automation yet
- small commits
- minimal diffs
- deterministic tests

After implementing Sprint 0, stop and summarize changed files, test results, and next recommended sprint.
```

---

## 20. Codex AGENTS.md draft

```markdown
# AGENTS.md

## Mission
Build a secure LinkedIn Career Automation MCP server for OgeonX AI.

## Non-negotiables
- Never commit secrets, tokens, cookies, client secrets, or `.env`.
- Never log secrets or OAuth tokens.
- Do not implement LinkedIn scraping.
- Do not claim Easy Apply API access.
- Public posting requires explicit approval.
- Keep changes small and testable.

## Stack
- Python 3.12+
- FastAPI
- Pydantic v2
- httpx
- pytest
- ruff
- SQLite local, PostgreSQL later

## Quality gate
Before finishing:
- run tests
- run ruff
- ensure `.env.example` has placeholders only
- ensure no secret-like values appear in committed files
```

---

## 21. Open questions

1. Final MCP transport path required by the ChatGPT Custom App UI: `/sse`, `/mcp`, or both.
2. Whether ChatGPT Custom App OAuth should authenticate to the MCP server directly, and the MCP server then separately performs LinkedIn OAuth, or whether the MCP server should expose LinkedIn as its authorization server. Recommended MVP: MCP server owns app auth and LinkedIn account linking internally.
3. Production host: Azure Container Apps or local tunnel first.
4. Database: SQLite MVP or PostgreSQL immediately.
5. Whether to include browser-assist worker in same repo or separate repo later. Recommended: separate later.

---

## 22. References

- OpenAI Apps SDK: `https://developers.openai.com/apps-sdk/`
- OpenAI Actions OAuth authentication: `https://developers.openai.com/api/docs/actions/authentication`
- MCP introduction: `https://modelcontextprotocol.io/docs/getting-started/intro`
- MCP authorization specification: `https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization`
- LinkedIn OpenID Connect: `https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/sign-in-with-linkedin-v2`
- LinkedIn Share on LinkedIn: `https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/share-on-linkedin`
- LinkedIn OAuth flow: `https://learn.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow`
- Apply with LinkedIn: `https://learn.microsoft.com/en-us/linkedin/talent/apply-with-linkedin/apply-with-linkedin`
