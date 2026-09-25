# Week 3 Submission - Individual Readiness Lab

## Student information

- Name: QIU GuanSheng
- Student ID: 21320398
- Repository: https://github.com/SkydenQQQ/HKUST-MAIE6000C-Skyden-Qiu
- Revision branch: `fix/week03-docker-verification`
- Required checkpoint tag: `w03-readiness`
- Commit SHA: resolve the published checkpoint with `git rev-parse 'w03-readiness^{commit}'`; record that exact SHA in Canvas.

## 1. What I changed

The bounded engineering change restores two existing integration tests that pytest silently omitted. The original filename `test_api_integration.py python` did not match pytest's Python test-file discovery pattern. Renaming it to `test_api_integration.py` restores collection without changing the test contents. The tests check that creating a case queues a pending job and that a healthy readiness endpoint returns `status: ok`.

The Week 3 Activity 5 prerequisite is also included: the Dockerfile installs `.[dev]` and copies `tests` into `/app/tests`, so tests can run inside the image.

Compared with the previous submitted checkpoint `1ca9e299f65ac96e47a2672e9cf36cea3e94707f`, this revision retains the same application code, tests, Dockerfile, migrations and CI configuration. It closes the previously disclosed Docker/PostgreSQL verification gap, records new evidence, updates this explanation, and removes redundant root-level copies of the submission files.

## 2. Files touched

Original bounded change, retained:
- `tests/integration/test_api_integration.py python` -> `tests/integration/test_api_integration.py`: filename-only rename.
- `Dockerfile`: test dependencies and test files in the image.

Resubmission evidence and documentation:
- `submissions/week03/README.md`: results and reproduction steps.
- `submissions/week03/docker-verification.txt`: successful Docker/PostgreSQL run on 25 September 2026.
- `submissions/week03/reverification-2026-09-25.txt`: fresh baseline/corrected native test comparison.
- `submissions/week03/environment-2026-09-25.txt`: final environment and resolved setup issues.
- `submissions/week03/verification.txt`: preserved historical native SQLite evidence from 22 September.
- Removed root duplicates `week03-README.md`, `week03-verification.txt`, `week03-changes.patch`; canonical material is in this directory.

## 3. How I verified it

### Actual results

The user executed the prepared verification script in Windows PowerShell 5.1 on 25 September 2026, 23:06-23:09 China time. The saved transcript was reviewed against the checks below. Docker Desktop 4.92.0, Docker Engine 29.8.0, Compose 5.5.1 and WSL 2.7.14 were used. The base image ran Python 3.11.16; PostgreSQL used `postgres:16-alpine`.

| Check | Observed result |
| --- | --- |
| Fresh native baseline vs corrected discovery/execution | 4 passed before; 6 passed after; Python 3.11.15 |
| Native Ruff | All checks passed |
| Compose configuration and image builds | Succeeded |
| Services | api, ai, db running and healthy; worker running (no dedicated worker healthcheck) |
| Container unit/integration collection and execution | 6 collected; 6 passed |
| Container Ruff on services and tests | All checks passed |
| Full suite against real Compose API | 7 passed, including the HTTP smoke test |
| API root, live, ready and Swagger docs | HTTP 200 |
| API metrics | HTTP 200; api_http_requests_total present |
| AI liveness | HTTP 200 |
| Login case | queued -> triaged; ai_label access; confidence 0.79 |
| Job | pending -> completed; attempts 1; error null |
| PostgreSQL rows | Case ('triaged', 'access'); job ('completed', 1, None) |
| Database migration | 20260713_0001 |
| Worker/AI logs | job_claimed, POST /triage, triage_completed, job_completed |

Traced case ID: `4f23e4fb-bf7e-46f2-92b7-9f1ec8e924a1`; job ID: `2`.

The six unit/integration tests use SQLite fixtures even when run inside Docker. The smoke test and separate SQL-checked trace exercise the real Compose API, worker, AI and PostgreSQL. Two dependency deprecation warnings were emitted; no test failed. The transcript's Windows account/machine header and local absolute paths are omitted or normalized for publication; verification results are unchanged.

### Reproduce

From the repository root, with Docker Desktop running:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose config --quiet
docker compose up --build -d --wait --wait-timeout 180
docker compose ps
docker compose run --rm --no-deps api pytest --collect-only -q tests/unit tests/integration
docker compose run --rm --no-deps api pytest -q tests/unit tests/integration
docker compose run --rm --no-deps api python -m ruff check services tests
docker compose run --rm --no-deps -e SMOKE_BASE_URL=http://api:8000 api pytest -q
```

Expected: six fixture tests pass, then all seven tests pass including the live-stack smoke test. API docs: `http://localhost:8000/docs`; AI liveness: `http://localhost:8100/health/live`. Use the actual ports from `.env` if changed.

Submit a synthetic login case, capture its returned case/job IDs, then retrieve `/cases/{id}` and `/jobs/{id}`. Confirm the final states above and inspect the same rows in PostgreSQL:

```powershell
docker compose exec db psql -U postgres -d maie6000c -c "SELECT id, status, ai_label FROM cases ORDER BY created_at DESC LIMIT 3;"
docker compose exec db psql -U postgres -d maie6000c -c "SELECT id, case_id, status, attempts, error FROM jobs ORDER BY id DESC LIMIT 3;"
docker compose logs --tail=30 worker ai
```

On this machine, Docker Hub initially timed out. The existing local HTTP proxy was configured in Docker Desktop and temporarily supplied to the build client as HTTP_PROXY/HTTPS_PROXY. Proxy values are machine-specific and are not application requirements. Internal hosts were excluded using NO_PROXY. The successful run did not delete any database volumes.

## 4. Known limitations or notes

- This change restores existing coverage, including some overlapping tests; it adds no application feature.
- The 22 September native SQLite evidence is historical; this revision adds the previously missing Docker/PostgreSQL evidence.
- Existing behavior remains: database outage can yield a readiness server error; failed jobs are not automatically retried. Outage/retry behavior was not separately tested in this run.
- No new remote CI result is claimed by the local Docker verification. Check the new PR's CI independently.
- Before Canvas resubmission, verify the remote w03-readiness tag refers to the new checkpoint, not the previous submission.
- Rollback by switching branches or reverting the change commit; no schema rollback or volume deletion is needed.
