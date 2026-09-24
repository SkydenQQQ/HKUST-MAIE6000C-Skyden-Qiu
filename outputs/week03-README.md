# Week 3 Submission - Individual Readiness Lab

## Student information

- Name: QIU GuanSheng
- Student ID: 21320398
- Repository: https://github.com/SkydenQQQ/HKUST-MAIE6000C-Skyden-Qiu
- Checkpoint tag: `w03-readiness`
- Commit SHA: resolve the final checkpoint with `git rev-parse 'w03-readiness^{commit}'`.

## 1. What I changed

The bounded change restores two existing integration tests that pytest silently omitted.
The file was named `test_api_integration.py python`, which does not match pytest's
normal Python test-file naming pattern. I renamed it to `test_api_integration.py`,
preserving its contents. The tests check that creating a case queues a job and
that a healthy readiness endpoint returns `status: ok`.

Before the change, the unit/integration command collected and passed four tests.
After the change, the same command collected and passed six tests, including both
tests from the renamed file. This closes a verification gap without changing
application behavior. The existing CI command also discovers the corrected file.

As the Activity 5 prerequisite, the Dockerfile now installs `.[dev]` and copies
`tests` into `/app/tests`. This makes the image contain pytest and the test files.
The Dockerfile change has been inspected, but not built in this environment.

## 2. Files touched

- `Dockerfile`: Activity 5 test dependencies and test files in the image.
- `tests/integration/test_api_integration.py python` ->
  `tests/integration/test_api_integration.py`: filename-only rename.
- `submissions/week03/README.md`: change, reproduction steps and limits.
- `submissions/week03/verification.txt`: captured verification outputs.

## 3. How I verified it

Verification date: 22 September 2026. Environment: Windows, Python 3.11.16,
project dependencies installed from `pyproject.toml` with the `dev` extra.
Baseline: clean clone of commit `d8bb9ad8fdca1b9b135bb32006566feedc257cb4`.
Local branch: `fix/week03-test-discovery`.

| Check | Result |
| --- | --- |
| Baseline unit/integration collection and execution | 4 collected; 4 passed |
| After rename, same collection and execution | 6 collected; 6 passed |
| `python -m ruff check .` | All checks passed |
| `python -m alembic upgrade head` on a fresh SQLite database | Upgrade to `20260713_0001` succeeded |
| `python -m pytest -q` with `SMOKE_BASE_URL` pointing to a running local API | 7 passed; no skipped smoke test |
| API `/`, `/health/live`, `/health/ready`, `/metrics` | HTTP 200; request counter present |
| AI `/health/live` | HTTP 200 |
| Case -> job -> worker -> AI -> persisted result | Case triaged as access; job completed in one attempt |

The unit/integration fixtures use SQLite. For the real HTTP smoke test, API, AI
and worker ran as separate native processes, sharing a fresh SQLite database
through the starter's existing SQLite support. This was not a Docker or
PostgreSQL run. Two dependency deprecation warnings were emitted; no test failed.
See `verification.txt` for outputs and the traced case/job IDs.

### Reproduce the unit and integration checks

From the repository root, use Python 3.11 in an activated virtual environment:

```powershell
python -m pip install -e ".[dev]"
python -m pytest --collect-only -q tests/unit tests/integration
python -m pytest -q tests/unit tests/integration
python -m ruff check .
```

Expected: six collected tests, six passing tests, and no Ruff errors.

### Reproduce the native local smoke check

Use a fresh database path and free ports. In three PowerShell terminals at the
repository root, activate the same Python 3.11 environment and set:

```powershell
$DbPath = Join-Path $PWD 'week03-local.db'
$env:DATABASE_URL = 'sqlite:///' + ($DbPath -replace '\\', '/')
$env:AI_SERVICE_URL = 'http://127.0.0.1:18100'
$env:WORKER_POLL_SECONDS = '0.2'
```

Terminal 1 (migrate first, then start the API):

```powershell
python -m alembic upgrade head
python -m uvicorn services.api.app.main:app --host 127.0.0.1 --port 18000
```

Terminal 2:

```powershell
python -m uvicorn services.ai.app.main:app --host 127.0.0.1 --port 18100
```

Terminal 3:

```powershell
python -m services.worker.app.main
```

In a fourth terminal with the same environment activated:

```powershell
$env:SMOKE_BASE_URL = 'http://127.0.0.1:18000'
python -m pytest -q
```

Expected: seven passing tests. Stop the three services with Ctrl+C afterwards.
The verification run used dynamically selected ports 27317 and 27318 instead.
Do not commit the local database or environment files.

### Docker/PostgreSQL follow-up (not executed)

After Docker Desktop is available, preserve any existing `.env`; create it from
`.env.example` only if absent. Run:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose config
docker compose build --no-cache api
docker compose run --rm --no-deps api sh -c "find /app/tests -maxdepth 3 -type f"
docker compose run --rm --no-deps api pytest -q tests/unit tests/integration
docker compose up --build -d
docker compose ps
```

For a configured API host port of 8000, with the local Python environment active:

```powershell
$env:SMOKE_BASE_URL = 'http://localhost:8000'
python -m pytest -q tests/smoke
```

Use the actual `API_HOST_PORT` if different. The six unit/integration tests still
use SQLite fixtures inside the image; the running-stack smoke test exercises
the Compose API, worker, AI service and PostgreSQL.

## 4. Known limitations or notes

- Docker Desktop is not installed. Image build, Compose startup and PostgreSQL
  behavior have not been verified. The native run is supplementary evidence.
- The rename restores existing coverage, some of which overlaps other tests;
  it does not add a feature or claim new application behavior.
- No service logic, database schema, migration or CI configuration was changed.
- The final commit SHA is recorded in the Canvas submission and is resolved
  by the checkpoint tag above. Record
  any additional checks only after they have really run.
- Rollback before merge: return to `main`. After merge, revert the change commit
  through normal Git history; no database rollback is needed for this change.
