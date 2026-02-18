# API Testing Demo (Postman/Newman + Docker + GitHub Actions)

![API Tests (Docker)](https://github.com/jessicazhong004/api-testing-demo/actions/workflows/api-docker.yml/badge.svg)
![API Tests (Newman)](https://github.com/jessicazhong004/api-testing-demo/actions/workflows/newman.yml/badge.svg)


A small API testing demo using a Postman collection executed by Newman.

- Provides a single entry point (`npm run test:api`) used locally, in Docker, and in CI.
- Includes two GitHub Actions workflows:
  - One runs Newman directly on the host.
  - One builds a Docker image and runs the tests inside a container, then uploads an HTML report as an artifact.

## Project Structure

```text
.
├── Dockerfile
├── .dockerignore
├── .github
│   └── workflows
│       ├── api-docker.yml      # Docker-based CI
│       └── newman.yml          # Newman-on-host CI
├── postman
│   ├── api_tests.postman_collection.json
│   └── local.postman_environment.json
└── reports
    └── report.html             # generated locally / in CI

```

## Prerequisites

- Node.js 18+ (Node 20 recommended)
- Docker Desktop (for the Docker-based workflow)

## Run Locally (npm script)

```bash
npm ci
npm run test:api
```
  - `npm run test:api` is the single entry point used locally, in Docker, and in CI.
  - The command generates `reports/report.html` using the `htmlextra` Newman reporter.

## Run in Docker

```bash
docker build -t api-testing-demo .
docker run --rm \
  -v "$PWD/reports:/app/reports" \
  api-testing-demo

```

This builds a Node-based image and runs `npm run test:api` inside the container, mounting `./reports` so the HTML report is written back to the host.

**Alternative: run Newman directly (no npm script)**

```bash
mkdir -p reports

npx newman run postman/api_tests.postman_collection.json \
  -e postman/local.postman_environment.json \
  -r htmlextra --reporter-htmlextra-export reports/report.html

open reports/report.html

```
**Environment Variables**

The collection uses `{{baseUrl}}` to build request URLs (e.g. `{{baseUrl}}/users`).

In Postman, ensure the environment includes:

*  `baseUrl = https://jsonplaceholder.typicode.com`

Note: the exported environment file is a full Postman environment JSON (not a single {` "key": "...", "value": "..." }` snippet).


**CI (GitHub Actions)**
* Docker workflow: `.github/workflows/api-docker.yml`
  * Build Docker image from `Dockerfile`
  * Run `npm run test:api` inside the container
  * Mount `reports/` and upload `reports/report.html` as an artifact

* Newman workflow: `.github/workflows/newman.yml`
  * Triggered on `push` and `pull_request`
  * Checks out the repo
  * Sets up Node (20)
  * Installs Newman + HTML reporter
  * Runs the collection with the environment file
  * Uploads `reports/report.html` as an artifact

**View the HTML Report (Artifact)**

GitHub repo **→ Actions →** open the latest successful run **→ Artifacts** → download `newman-report` → open `report.html`.

**Notes / Common Issues**
* CI only reads files committed to the GitHub repo.
  If collection/environment files are untracked, the runner will fail with `ENOENT`.
* If requests show `http:///...`, it usually means `baseUrl` is empty/mismatched or the request URL is not using `{{baseUrl}}`.
* If you rename/move files under `postman/`, update the paths in:
  * `.github/workflows/newman.yml`
  * `.github/workflows/api-docker.yml`
  * `package.json` (`test:api` script)