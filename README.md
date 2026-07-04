# API Automation with Postman, Newman & GitHub Actions

A personal QA automation project that takes a Postman API test collection and turns it into a fully automated CI/CD pipeline — running on every push, on a nightly schedule, with automatic email notifications and both HTML and JUnit reports.

**Sample API under test:** [DummyJSON](https://dummyjson.com) — a free public mock API (no key required), used to demonstrate Auth flows, CRUD operations, and request chaining.

---

## ✨ What this project demonstrates

- ✅ Postman collection with structured folders: **Auth**, **Products (CRUD)**, **Users**, **Todos**
- ✅ Token capture and chaining between requests (login → authenticated calls)
- ✅ Test scripts (`pm.test`) validating status codes, response shape, and values
- ✅ Newman — running the same collection from the CLI
- ✅ HTML reports (human-readable) and JUnit XML reports (CI-readable)
- ✅ GitHub Actions pipeline — triggered on push, pull request, manual dispatch, and a nightly cron schedule
- ✅ Secrets handled properly — no credentials committed to the repo, injected via GitHub Actions Secrets
- ✅ Email notifications on every run (pass or fail), with the HTML report attached

---

## 📁 Project structure

```
api-automation/
├── collections/
│   └── collection.json              # Postman collection
├── environments/
│   └── environment.json              # Environment variables (credentials blanked out)
├── .github/
│   └── workflows/
│       └── api-tests.yml             # CI/CD pipeline definition
├── reports/                           # Generated at runtime (git-ignored)
└── .gitignore
```

---

## 🚀 Running locally

### Prerequisites
- [Postman](https://www.postman.com/downloads/) (optional, for editing the collection)
- Node.js 18+ and npm
- [Newman](https://www.npmjs.com/package/newman)

### Install Newman + reporters

```bash
sudo npm install -g newman newman-reporter-htmlextra newman-reporter-junitfull
```

### Run the collection

```bash
newman run collections/collection.json \
  -e environments/environment.json \
  --env-var username=<your-username> \
  --env-var password=<your-password> \
  -r cli,htmlextra,junitfull \
  --reporter-htmlextra-export reports/report.html \
  --reporter-junitfull-export reports/report.xml
```

View the report:

```bash
xdg-open reports/report.html
```

> The `environments/environment.json` file ships with `username`/`password` blank intentionally — pass real values via `--env-var` at runtime, or fill them locally in Postman (never commit real credentials).

---

## 🔄 CI/CD Pipeline

The workflow at [`.github/workflows/api-tests.yml`](.github/workflows/api-tests.yml) runs automatically:

| Trigger | When |
|---|---|
| `push` | Every push to `main` |
| `pull_request` | Every PR targeting `main` |
| `workflow_dispatch` | Manually, from the Actions tab |
| `schedule` | Nightly at 02:00 UTC |

On each run it:
1. Installs Newman and reporters
2. Runs the full collection against DummyJSON
3. Generates HTML + JUnit reports
4. Uploads the HTML report as a downloadable artifact
5. Publishes a pass/fail summary directly in the Actions UI
6. Sends an email (pass or fail) with the report attached

### Required GitHub Secrets

| Secret | Purpose |
|---|---|
| `DUMMY_USERNAME` | Login username for the test API |
| `DUMMY_PASSWORD` | Login password for the test API |
| `MAIL_USERNAME` | Sender Gmail address |
| `MAIL_PASSWORD` | Gmail app password (not your real password) |
| `MAIL_TO` | Recipient email for run notifications |

Set these under **Settings → Secrets and variables → Actions**.

---

## 🐛 Notable issues hit & fixed

| Issue | Fix |
|---|---|
| `ENOENT: no such file` running Newman | Filenames had spaces — wrapped paths in quotes |
| `404` on Update/Delete Product | Was targeting a fake ID from a simulated Create response — fixed to use a real product ID from a GET request |
| `Failed to create checks using the provided token` in Actions | Default `GITHUB_TOKEN` is read-only — added explicit `permissions: checks: write` |

---

## 📌 Roadmap / possible next steps

- [ ] Data-driven testing with multiple input datasets (`-d data.json`)
- [ ] Separate staging/production environment files run via a build matrix
- [ ] Slack notifications alongside email

---

## 🧰 Tech stack

`Postman` · `Newman` · `Node.js` · `GitHub Actions` · `DummyJSON`

---

## 📄 License

This is a personal learning/portfolio project. Feel free to fork and adapt.
