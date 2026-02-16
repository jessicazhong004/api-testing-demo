# API Testing Demo (Postman + Newman + GitHub Actions)

Small API testing demo using Postman collections executed by Newman.
Includes GitHub Actions CI that runs on every push/PR and uploads an HTML report as an artifact.

## Project Structure
```text
.
├── .github/workflows/newman.yml
└── postman
├── api_tests.postman_collection.json
└── local.postman_environment.json
```

## Prerequisites

- Node.js 18+ (Node 20 recommended)
- Newman (optional locally if you use `npx`)

## Run Locally

From repo root:

```bash
npx newman run postman/api_tests.postman_collection.json \
  -e postman/local.postman_environment.json
  ```
(Optional) Generate an HTML report locally:
```bash
mkdir -p reports
npm i -g newman newman-reporter-htmlextra
```
newman run postman/api_tests.postman_collection.json \
```bash
  -e postman/local.postman_environment.json \
  -r htmlextra --reporter-htmlextra-export reports/report.html
  ```

Then open:
```bash
open reports/report.html
```
**Environment Variables**
The collection uses `{{baseUrl}}` to build request URLs (e.g. `{{baseUrl}}/users`).

Make sure `postman/local.postman_environment.json` includes:

* `baseUrl: https://jsonplaceholder.typicode.com`

Example:
```json
{
  "key": "baseUrl",
  "value": "https://jsonplaceholder.typicode.com",
  "enabled": true
}
```
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

**Notes**
* CI reads files from the GitHub repo. If you rename/move collection/environment files, update the paths in `newman.yml`.

* If requests show `http:///...`, it usually means `baseUrl` in the exported environment JSON is empty or mismatched.