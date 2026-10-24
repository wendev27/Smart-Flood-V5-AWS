# SmartFlood V5 AWS Migration

SmartFlood V5 is the AWS migration destination for the SmartFlood V4.1
application. Phase 1 imports the application source and records its verified
runtime boundaries. AWS infrastructure, containers, CI/CD, and deployment are
separate later phases.

## Application architecture

The imported application has three services:

| Service  | Implementation                      |   Local port | Production entry point                                                        |
| -------- | ----------------------------------- | -----------: | ----------------------------------------------------------------------------- |
| Frontend | Next.js / React / TypeScript        |         3000 | `npm start`                                                                   |
| API      | Next.js route handlers / TypeScript |         5000 | `npm start` (script binds port 5000)                                          |
| AI       | FastAPI / Python                    | 8000 locally | `gunicorn app.main:app -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:$PORT` |

The API is not an Express service. It owns the application route handlers,
authentication/session handling, RBAC checks, MongoDB access, Supabase access,
reporting, and the bridge to the AI service.

The request path is intentionally preserved:

```text
Browser
  |
  | /api/* requests
  v
Frontend Next.js rewrite
  |
  v
Backend/api Next.js route handlers
  |
  +--> MongoDB Atlas
  +--> Supabase
  +--> Backend/ai FastAPI
```

The frontend rewrite is configured by `NEXT_PUBLIC_API_URL` in
`Frontend/next.config.ts`. The browser should use the frontend `/api/*`
boundary; direct public access to the API or AI service is not required by the
verified application flow.

## Health endpoints

The verified liveness endpoints are:

| Service  | Endpoint                    | Behavior                                                                        |
| -------- | --------------------------- | ------------------------------------------------------------------------------- |
| Frontend | No dedicated endpoint found | Use the application root only after deployment health-check design is reviewed. |
| API      | `/api/health`               | Static response; currently retains its existing V3 wording.                     |
| AI       | `/health`                   | Static service status response.                                                 |

These endpoints are liveness checks. They do not verify MongoDB, Supabase,
AI-service, or weather-provider readiness. The API health response is
deliberately unchanged in this phase; correcting its stale message is a
separate application change.

## Repository layout

```text
Frontend/                 Next.js web application
Backend/api/              Next.js API route-handler application
Backend/ai/               FastAPI AI service
supabase/migrations/      Existing migration references; not applied here
README.md                 Migration and runtime documentation
```

Local dependencies, generated output, local environments, credentials, and
repository metadata are intentionally not part of this repository. In
particular, `.env` files, `node_modules`, `.next`, `.venv`, Python caches, and
the V4.1 `.git` directory are excluded.

## Local development

Frontend:

```bash
cd Frontend
npm install
npm run dev
```

API:

```bash
cd Backend/api
npm install
npm run dev
```

AI service:

```bash
cd Backend/ai
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt pytest
.venv/bin/python -m uvicorn app.main:app --reload --port 8000
```

Use the tracked `.env.example` files as templates. Do not copy local `.env`
files or place production credentials in the repository.

## Environment variables

### Frontend

Public/browser configuration:

- `NEXT_PUBLIC_API_URL`
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `NODE_ENV`

Server-side provider credentials used by frontend weather routes:

- `TOMORROW_API_KEY`
- `OPENWEATHER_API_KEY`

The weather keys must not be renamed with a `NEXT_PUBLIC_` prefix.

### API

Private runtime configuration:

- `MONGODB_URI`
- `MONGODB_DB`
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
- `SMARTFLOOD_SESSION_SECRET`
- `AI_BACKEND_URL`
- `CORS_ORIGINS`
- `CAMPAIGN_QR_ENCRYPTION_KEY`
- `NODE_ENV`
- `PORT`

`CAMPAIGN_QR_ENCRYPTION_KEY` must encode exactly 32 bytes. Production should
set the dedicated `SMARTFLOOD_SESSION_SECRET`; the existing service-role
fallback remains application behavior and is not changed here.

### AI service

Private runtime configuration:

- `MONGODB_URI`
- `MONGODB_DB`
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
- `CORS_ORIGINS`
- `PORT`

MongoDB Atlas and Supabase remain external managed dependencies. No database
schema, authentication/RBAC behavior, or API contract is changed by the AWS
migration documentation.

## AWS migration direction

The intended target is AWS in `ap-southeast-1`:

- Amazon ECR repositories for frontend, API, and AI images.
- ECS/Fargate services for the three application services.
- An internet-facing ALB for the frontend only.
- Private service-to-service connectivity from frontend to API and API to AI.
- CloudWatch logs with explicit retention.
- Secrets Manager or Parameter Store for private runtime values.
- Terraform for staging infrastructure.
- GitHub Actions with restricted AWS OIDC for image publishing and deployment.

These resources are not created by Phase 1.

## Phase status

### Completed in Phase 1

- Imported the required frontend, API, AI, and Supabase migration source.
- Preserved service lockfiles and application configuration.
- Added the V4.1 root `.gitignore`.
- Corrected this README to match verified frameworks, ports, entry points,
  request flow, health endpoints, dependencies, and environment variables.

### Not included in Phase 1

- Dockerfiles or container builds.
- Terraform or AWS resources.
- GitHub Actions deployment workflows.
- Application behavior, health responses, schemas, authentication, or API
  contract changes.

## Validation policy

Phase 1 validates repository contents and exclusions only. Builds, tests,
container startup, and service-to-service communication belong to later
phases and must not be reported as successful until actually executed.
