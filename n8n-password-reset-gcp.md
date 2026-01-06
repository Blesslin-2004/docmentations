# n8n Self-Hosted Password Reset (GCP VM)

This document explains how to reset a forgotten password for a **self-hosted n8n instance** running on a **GCP VM using Docker**.

---

## ✅ Method Used: n8n CLI Password Reset (Recommended)

This is the **safest and officially supported** way to reset the password without touching the database or volumes.

---

## Prerequisites

- SSH access to the GCP VM
- n8n running inside a Docker container
- Docker installed on the VM

---

## Step-by-Step Guide

### 1. SSH into the GCP VM
```bash
gcloud compute ssh <VM_NAME> --zone <ZONE>
```

---

### 2. List running Docker containers
```bash
docker ps
```

Identify the container running n8n.  
Example output:
```text
CONTAINER ID   IMAGE        NAMES
abc123         n8nio/n8n    n8n
```

---

### 3. Enter the n8n container
```bash
docker exec -it <n8n_container_name> sh
```

Example:
```bash
docker exec -it n8n sh
```

---

### 4. Reset the n8n user password
Inside the container, run:
```bash
n8n user-management:reset
```

You will be prompted to:
- Enter the **email address**
- Set a **new password**

Once completed, the password is updated successfully.

---

### 5. Exit the container
```bash
exit
```

---

### 6. (Optional but Recommended) Restart the n8n container
```bash
docker restart <n8n_container_name>
```

---

## ✅ Result

You can now log in to the n8n web UI using the **new password**.

---

## Notes

- This method works for both **SQLite** and **PostgreSQL** databases.
- No data, workflows, or credentials are affected.
- No volumes or containers are deleted.

---

## ⚠️ What NOT to Do

- ❌ Do not delete the database
- ❌ Do not remove Docker volumes
- ❌ Do not recreate containers unnecessarily

---

## Reference

Official n8n CLI command:
```bash
n8n user-management:reset
```

---

**Documented after successful recovery on GCP VM**
