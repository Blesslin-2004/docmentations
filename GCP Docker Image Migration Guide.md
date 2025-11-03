 # 🧭 GCP Docker Image Migration Guide

This guide explains **how to migrate all Docker images** from one GCP Artifact Registry project to another — step by step, without exposing sensitive details.

---

## ⚙️ Environment Setup

### Old Project (Source)

* **Project ID:** `<OLD_PROJECT_ID>`
* **Region:** `<REGION>`
* **Repositories:** `<OLD_REPO_PATHS>` (e.g., `team1/app`, `team1/backend`, etc.)

### New Project (Destination)

* **Project ID:** `<NEW_PROJECT_ID>`
* **Region:** `<REGION>`
* **Repositories:** `<NEW_REPO_PATHS>` (same names as old or renamed as needed)

### Accounts

* **Old project account:** `<OLD_ACCOUNT_EMAIL>`
* **New project account:** `<NEW_ACCOUNT_EMAIL>`

---

## 🧩 Step 1 — Verify Login and Set Active Account

```bash
gcloud auth list
gcloud config set account <NEW_ACCOUNT_EMAIL>
```

Ensures the **active account** (`*`) is the one with access to the **new project**.

---

## 🧩 Step 2 — Verify or Create Target Repositories

Check existing repositories:

```bash
gcloud artifacts repositories list --project=<NEW_PROJECT_ID> --location=<REGION>
```

Create missing ones:

```bash
gcloud artifacts repositories create <REPO_NAME> \
  --repository-format=docker \
  --location=<REGION> \
  --project=<NEW_PROJECT_ID> \
  --description="Docker repo for <APP_NAME>"
```

---

## 🧩 Step 3 — Assign Artifact Registry Permissions

Grant write access to your new project account:

```bash
gcloud projects add-iam-policy-binding <NEW_PROJECT_ID> \
  --member="user:<NEW_ACCOUNT_EMAIL>" \
  --role="roles/artifactregistry.writer"
```

---

## 🧩 Step 4 — Configure Docker Authentication

```bash
gcloud auth configure-docker <REGION>-docker.pkg.dev
```

---

## 🧩 Step 5 — List All Docker Images in the Old Project

```bash
gcloud artifacts docker images list <REGION>-docker.pkg.dev/<OLD_PROJECT_ID>/<REPO_PATH>
```

---

## 🧩 Step 6 — Pull Docker Images From Old Project

```bash
docker pull <REGION>-docker.pkg.dev/<OLD_PROJECT_ID>/<REPO_PATH>@sha256:<DIGEST>
```

Confirm images:

```bash
docker images
```

---

## 🧩 Step 7 — Tag and Push to New Project

```bash
docker tag <REGION>-docker.pkg.dev/<OLD_PROJECT_ID>/<REPO_PATH>@sha256:<DIGEST> <REGION>-docker.pkg.dev/<NEW_PROJECT_ID>/<REPO_PATH>:<SHORT_DIGEST> && docker push <REGION>-docker.pkg.dev/<NEW_PROJECT_ID>/<REPO_PATH>:<SHORT_DIGEST>
```

Example: Replace `<SHORT_DIGEST>` with first 8 characters of the SHA hash.

---

## 🧩 Step 8 — Verify in New Project

```bash
gcloud artifacts docker images list <REGION>-docker.pkg.dev/<NEW_PROJECT_ID>/<REPO_PATH>
```

---

## 🚨 Troubleshooting

**Error: `Permission denied (uploadArtifacts)`**
→ Add IAM role `roles/artifactregistry.writer` and confirm repository exists.

**Error: `not found` when pulling image**
→ Run the list command again to verify digest.

---

## ✅ Verification Checklist

| Step | Description                             | Verified |
| ---- | --------------------------------------- | -------- |
| 1    | Active account is `<NEW_ACCOUNT_EMAIL>` | ☐        |
| 2    | Repositories exist in new project       | ☐        |
| 3    | IAM permissions set                     | ☐        |
| 4    | Docker auth configured                  | ☐        |
| 5    | Images pulled successfully              | ☐        |
| 6    | Images pushed successfully              | ☐        |
| 7    | Verified in destination registry        | ☐        |

---

**Author:** Blesslin
**Date:** November 2025
**Purpose:** Secure and complete GCP Docker image migration.
