# Cloud Infrastructure Lab — Phase 3

## CI/CD Pipeline with GitHub Actions

This phase automates the build process of the **White** backend using GitHub Actions.

Instead of building Docker images manually on the servers, every build now runs in a clean, reproducible environment: the code is tested against a real PostgreSQL database, packaged into a Docker image, and pushed to Docker Hub with a unique, traceable tag.

---

## Pipeline Architecture

![Pipeline Architecture](images/phase-3-architecture.png)

```text
Manual Trigger (workflow_dispatch)
        ↓
GitHub Actions
        ↓
┌──────────────────────────────┐
│ Job 1: Test                  │
│  • PostgreSQL service        │
│  • Install dependencies      │
│  • Run pytest                │
└──────────────┬───────────────┘
               │ (only if tests pass)
               ↓
┌──────────────────────────────┐
│ Job 2: Build & Push          │
│  • Log in to Docker Hub      │
│  • Build Docker image        │
│  • Push image (tag = SHA)    │
└──────────────┬───────────────┘
               ↓
          Docker Hub
   mahmoudeng/white-backend:<commit-sha>
```

---

## Pipeline Overview

| Item             | Value                                    |
| ---------------- | ---------------------------------------- |
| Workflow name    | `White CI/CD`                            |
| Trigger          | `workflow_dispatch` (manual run)         |
| Runner           | `ubuntu-latest`                          |
| Jobs             | `test` → `build`                         |
| Registry         | Docker Hub                               |
| Image            | `mahmoudeng/white-backend`               |
| Image tag        | Git commit SHA (`${{ github.sha }}`)     |

### Repository

The workflow runs in the backend repository:

[MHM-X/flask-server](https://github.com/MHM-X/flask-server)

The workflow file is located at:

```text
.github/workflows/<workflow-file>.yml
```

---

## Trigger

```yaml
on:
  workflow_dispatch:
```

The pipeline is started manually from the **Actions** tab on GitHub.

This keeps full control over when an image is built and published while the pipeline is still being developed.

---

## Job 1 — Test

The first job verifies that the application works before any image is built.

### PostgreSQL service container

Instead of mocking the database, the job starts a real PostgreSQL container alongside the runner:

```yaml
services:
  postgres:
    image: postgres:18

    env:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
      POSTGRES_DB: white_test_db

    ports:
      - 5432:5432

    options: >-
      --health-cmd="pg_isready -U test_user -d white_test_db"
      --health-interval=5s
      --health-timeout=5s
      --health-retries=5
```

The health check makes sure PostgreSQL is ready to accept connections before the tests start.

### Environment variables

```yaml
env:
  DATABASE_URL: postgresql+psycopg://test_user:test_password@localhost:5432/white_test_db
  PYTHONPATH: .
```

* `DATABASE_URL` points the application to the temporary test database.
* `PYTHONPATH` allows the tests to import the application modules from the project root.

The credentials here belong to a throwaway database that only exists for the duration of the job.

### Steps

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v6

  - name: Set up Python
    uses: actions/setup-python@v6
    with:
      python-version: "3.12"

  - name: Install dependencies
    run: pip install -r requirements.txt

  - name: Run tests
    run: pytest
```

| Step                 | Purpose                                       |
| -------------------- | --------------------------------------------- |
| Checkout repository  | Downloads the source code to the runner       |
| Set up Python        | Installs Python 3.12 (same version as the Docker image) |
| Install dependencies | Installs the packages from `requirements.txt` |
| Run tests            | Runs the test suite against PostgreSQL        |

If any test fails, the job fails and the pipeline stops. No image is built.

---

## Job 2 — Build & Push

```yaml
build:
  needs: test
  runs-on: ubuntu-latest
```

`needs: test` makes this job run **only after** the test job succeeds.

### Steps

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v6

  - name: Log in to Docker Hub
    uses: docker/login-action@v3
    with:
      username: ${{ secrets.DOCKERHUB_USERNAME }}
      password: ${{ secrets.DOCKERHUB_TOKEN }}

  - name: Build Docker image
    run: |
      docker build \
        -t mahmoudeng/white-backend:${{ github.sha }} \
        .

  - name: Push Docker image
    run: |
      docker push mahmoudeng/white-backend:${{ github.sha }}
```

| Step                | Purpose                                                 |
| ------------------- | ------------------------------------------------------- |
| Checkout repository | Downloads the source code and the `Dockerfile`          |
| Log in to Docker Hub| Authenticates using repository secrets                  |
| Build Docker image  | Builds the backend image from the `Dockerfile`          |
| Push Docker image   | Publishes the image to Docker Hub                       |

---

## Image Tagging Strategy

Each image is tagged with the Git commit SHA:

```text
mahmoudeng/white-backend:a81f32c9e7...
```

Compared with using a tag like `latest`, this gives:

* **Traceability:** every image maps to the exact commit it was built from.
* **Immutability:** a tag always points to the same code.
* **Safe rollbacks:** going back to a previous version means using an older tag.
* **No ambiguity:** servers never pull a different image under the same tag.

---

## Secrets

Docker Hub credentials are stored as encrypted repository secrets and are never written in the workflow file.

| Secret               | Description                    |
| -------------------- | ------------------------------ |
| `DOCKERHUB_USERNAME` | Docker Hub account username    |
| `DOCKERHUB_TOKEN`    | Docker Hub access token        |

They are configured under:

```text
Repository → Settings → Secrets and variables → Actions
```

An access token is used instead of the account password so it can be revoked without affecting the account.

---

## Connection to Phase 2

The image produced by this pipeline is the same image that the Flask servers pull in Phase 2.

The `IMAGE_TAG` variable in Phase 2 is set to the commit SHA generated by this pipeline:

```bash
export IMAGE_TAG=<COMMIT_SHA>
sudo docker compose pull backend
sudo -E docker compose up -d backend
```

```yaml
image: mahmoudeng/white-backend:${IMAGE_TAG}
```

The full flow is now:

```text
Code change
    ↓
GitHub Actions (test → build → push)
    ↓
Docker Hub
    ↓
Flask Server 1 / Flask Server 2
    (docker compose pull + up)
```

---

## Running the Pipeline

1. Open the backend repository on GitHub.
2. Go to the **Actions** tab.
3. Select the **White CI/CD** workflow.
4. Click **Run workflow**.
5. Select the branch and confirm.
6. Watch the `test` and `build` jobs run.
7. Copy the commit SHA and use it as `IMAGE_TAG` on the servers.

---

## Technology Stack

| Layer              | Technology                   |
| ------------------ | ---------------------------- |
| CI/CD              | GitHub Actions               |
| Test Database      | PostgreSQL 18 (service container) |
| Test Framework     | pytest                       |
| Language           | Python 3.12                  |
| Containerization   | Docker                       |
| Image Registry     | Docker Hub                   |
| Secrets Management | GitHub Actions Secrets       |

> This phase automates testing and image publishing, completing the path from a code change to a ready-to-deploy image.
