# API Testing Demo (Postman + Newman + GitHub Actions)

A small API testing demo using a Postman collection executed by Newman.
Includes GitHub Actions CI that runs on every push/PR and uploads an HTML report as a CI artifact.

## Project Structure

```text
.
├── .github/workflows/newman.yml
├── postman
│   ├── api_tests.postman_collection.json
│   └── local.postman_environment.json
└── reports
    └── report.html   (generated locally / in CI)

```

## Prerequisites

- Node.js 18+ (Node 20 recommended)

## Run Locally

From repo root:

```bash
npx newman run postman/api_tests.postman_collection.json \
  -e postman/local.postman_environment.json
  ```

**Generate an HTML Report Locally (Optional)**

```bash
mkdir -p reports
npm i -g newman newman-reporter-htmlextra

newman run postman/api_tests.postman_collection.json \

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
Workflow: `.github/workflows/newman.yml`

Triggers:

* `push`

* `pull_request`

What CI does:

1. Checks out the repo

2. Sets up Node (20)

3. Installs Newman + HTML reporter

4. Runs the collection with the environment file

5. Uploads `reports/report.html` as an artifact

**View the HTML Report (Artifact)**

GitHub repo **→ Actions →** open the latest successful run **→ Artifacts** → download `newman-report` → open `report.html`.

**Notes / Common Issues**
* CI only reads files committed to the GitHub repo. If collection/environment files are untracked, the runner will fail with `ENOENT`.

* If requests show `http:///...`, it usually means `baseUrl` is empty/mismatched or the request URL is not using `{{baseUrl}}`.

* If you rename/move files under `postman/`, update the paths in `.github/workflows/newman.yml`.