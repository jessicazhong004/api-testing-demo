# API Testing Demo (Postman + Newman + Docker)

![API Tests (Docker)](https://github.com/jessicazhong004/api-testing-demo/actions/workflows/api-docker.yml/badge.svg)
![API Tests (Newman)](https://github.com/jessicazhong004/api-testing-demo/actions/workflows/newman.yml/badge.svg)

## What is this?

Small API testing demo that runs a Postman collection with Newman:
- locally with `npm run test:api`
- inside a Docker container
- in GitHub Actions CI

## Why it matters

- Uses a **single entry point** (`npm run test:api`) reused in local, Docker, and CI.
- Keeps tests in a **consistent environment** (Docker) to reduce “works on my machine” issues.
- Produces a **HTML report** and uploads it as a CI artifact for easy inspection.

## How to run

### Local

```bash
npm ci
npm run test:api
```
This will generate a HTML report at:
```text
reports/report.html
```
**Docker**
```bash
docker build -t api-testing-demo .
docker run --rm \
  -v "$PWD/reports:/app/reports" \
  api-testing-demo
```
The container run will also write the HTML report to:
```text
reports/report.html
```
**CI (GitHub Actions)**
Two workflows live in `.github/workflows`:
* `newman.yml` – installs Newman globally and runs the Postman collection on the host.
* `api-docker.yml` – builds the Docker image and runs the tests inside a container, mounting reports/.
Both workflows upload the same HTML report as an artifact named:
```text
newman-report
```
You can find it under:

  GitHub → Actions → select a run → Artifacts → `newman-report`
**What you'll see**
* Locally: `npm run test:api` prints Newman’s summary in the terminal and writes `reports/report.html`.
* In CI: each workflow uploads the HTML report (reports/report.html) as the newman-report artifact.