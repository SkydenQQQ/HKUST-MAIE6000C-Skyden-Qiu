# Week 3 Submission - Individual Readiness Lab

## Student information

- Name: QIU GuanSheng
- Student ID: 21320398
- Repository: https://github.com/SkydenQQQ/HKUST-MAIE6000C-Skyden-Qiu
- Revision branch: `fix/week03-docker-verification`
- Required checkpoint tag: `w03-readiness`
- Existing remote checkpoint: `1ca9e299f65ac96e47a2672e9cf36cea3e94707f` (previous submission; does not include this revision).
- Status: **pending Docker/PostgreSQL verification and authenticated publication**. Do not submit this revision as a completed Docker readiness checkpoint yet.

## 1. What I changed

The bounded change restores two existing integration tests that pytest silently omitted. The original filename, `test_api_integration.py python`, does not match pytest's Python test-file discovery pattern. Renaming it to `test_api_integration.py` restores collection without changing its contents.

The restored tests verify that creating a case queues a pending job and that a healthy readiness endpoint returns `status: ok`. The baseline unit/integration suite has four tests; the corrected suite has six. This improves verification coverage without changing application behavior.

The Week 3 Activity 5 prerequisite is also retained: the Dockerfile installs `.[dev]` and copies `tests` into `/app/tests`, allowing pytest to run inside the image. Image build success is not claimed until Docker evidence is actually recorded.

This revision corrects documentation, adds an explicit AI Use Statement, and removes redundant root-level submission copies. The canonical submission lives under `submissions/week03/`.

## 2. Files touched

- `tests/integration/test_api_integration.py python` -> `tests/integration/test_api_integration.py`: filename-only rename.
- `Dockerfile`: install test dependencies and include test files.
- `submissions/week03/README.md`: explanation, actual status and reproduction steps.
- `submissions/week03/reverification-2026-09-25.txt`: fresh local test evidence.
- `submissions/week03/verification.txt`: preserved historical native verification, dated 22 September 2026; not Docker evidence.
- Root-level `week03-README.md`, `week03-verification.txt` and `week03-changes.patch`: redundant previous delivery copies removed.

## 3. How I verified it

### Local baseline and corrected tests

Current main, commit `2891ac83fbc48be9d18a3e021f72bb0eb0a00494`, restores the initial project tree. It was cloned separately for the baseline comparison. The revision retains the filename fix already in `submission/week03`.

Use Python 3.11 (the project requires `>=3.11,<3.12`) in a virtual environment:

```powershell
python -m pip install -e ".[dev]"
python -m pytest --collect-only -q tests/unit tests/integration
python -m pytest -q tests/unit tests/integration
python -m ruff check .
```

Expected discovery: four tests on unmodified main, six after the rename. On 25 September, Python 3.11.15 reproduced four collected/passing baseline tests and six collected/passing corrected tests; Ruff passed. Two dependency deprecation warnings were reported. Actual command outputs and exit codes are in `reverification-2026-09-25.txt`. Fixtures explicitly use SQLite; these checks cannot establish that Docker, PostgreSQL or the asynchronous worker stack works.

The older `verification.txt` records a native API/AI/worker smoke run using SQLite on 22 September. It is historical evidence, not a new run or equivalent to Compose verification.

### Required Docker/PostgreSQL verification - pending

On 25 September Git 2.40.0 and VS Code were found. The user then installed Docker Desktop 4.92.0. Its actual dashboard reports `Virtualization support not detected` and `Engine stopped`; `wsl --status` reports that WSL is not installed. Docker is installed, but the engine and the following checks are not yet ready.

After WSL 2 and Docker Desktop are installed and the engine is running:

```powershell
git --version
docker --version
docker compose version
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose config --quiet
docker compose up --build -d --wait --wait-timeout 180
docker compose ps
docker compose run --rm --no-deps api pytest --collect-only -q tests/unit tests/integration
docker compose run --rm --no-deps api pytest -q tests/unit tests/integration
docker compose run --rm --no-deps -e SMOKE_BASE_URL=http://api:8000 api pytest -q
```

Expected: `db`, `ai`, `api`, `worker` running; six unit/integration tests pass; all seven tests pass when the smoke test targets the real Compose API. Fixture tests still use SQLite inside Docker; the smoke test exercises the API/worker/AI/PostgreSQL stack.

Open `http://localhost:8000/docs` (or the actual `API_HOST_PORT` in `.env`). Check `/`, `/health/live`, `/health/ready`, `/metrics`, and `http://localhost:8100/health/live`. Submit a synthetic login case and capture its case/job IDs. Confirm `triaged`/`access` and `completed` with `attempts: 1`, then check those IDs in PostgreSQL and worker/AI logs.

Save actual outputs as `docker-verification.txt` only after execution. The separately supplied `Verify-Docker-Lab.ps1` automates these checks and copies its transcript here only on success. Its syntax is checked; runtime behavior is unverified until Docker is available.

### Git publication

The new local branch is `fix/week03-docker-verification`. The user initially reported a GitHub login restriction, then confirmed browser login was restored. Command-line push authentication still did not succeed in this session (`cannot spawn sh` / credentials unavailable). Public reads work; browser login alone has not established a usable Git push session.

After Docker verification passes, update this README with actual results, commit the evidence, push the branch and review a PR against main. Merge the verified change before publishing the final checkpoint. The existing `w03-readiness` tag must not be silently overwritten: confirm its replacement with the repository owner, preserve the previous commit reference, and replace it only when the corrected checkpoint is ready.

Resolve the final tag with `git rev-parse 'w03-readiness^{commit}'`. Verify the remote tag resolves to the same commit before pasting the Canvas entry. A local commit or zip does not satisfy the pushed-tag requirement.

## 4. Known limitations or notes

- Docker image build, Compose startup and PostgreSQL end-to-end behavior remain unverified. This is a real readiness gap.
- The rename restores existing coverage, some overlapping; it adds no application feature.
- No application source, migration or CI workflow was changed.
- Existing behavior: readiness raises a server error if the database is unavailable; failed jobs are not retried automatically.
- Rollback: switch away from this branch, or revert the relevant commit after merge. No schema rollback is needed. Do not delete database volumes to undo this change.

## 5. AI Use Statement

- Tool: OpenAI Codex.
- Purpose: compare course instructions with the repository, review the existing filename correction, assist with setup, execute verification, and draft documentation and a Docker verification helper.
- Materially assisted areas: Dockerfile/test-discovery review, verification evidence, this README, the separate helper and Canvas draft.
- Checks and limitations: Codex checked that the renamed test content is unchanged and distinguished native SQLite results from Docker/PostgreSQL evidence. Docker validation and remote publication are not claimed.
- Rejected approach: treating a native SQLite run as proof of Docker/PostgreSQL readiness, or tagging unverified work as complete.
- Student review: this statement does not assert that the student personally ran tool-executed commands. The student must review and understand the change and update final verification/publication status before submission.
