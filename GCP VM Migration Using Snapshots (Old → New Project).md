# 🚀 GCP VM Migration Using Snapshots (Old → New Project)

## 🧩 Overview
This document explains how to migrate Google Compute Engine (GCE) VMs from one project to another using disk snapshots.  
This method allows full migration with minimal downtime and no detachment of disks during production runtime.

---

## 🏗️ Environment Details

| Parameter | Description |
|------------|-------------|
| Old Project ID | `<OLD_PROJECT_ID>` |
| New Project ID | `<NEW_PROJECT_ID>` |
| Region | `<REGION>` |
| VMs Migrated | `<VM_1> (<VM_1_TYPE>)`, `<VM_2> (<VM_2_TYPE>)` |

---

## 1️⃣ Create Snapshots in the Old Project

Run these commands from the **old project context**:

```bash
gcloud config set project <OLD_PROJECT_ID>
gcloud compute disks snapshot <VM_1> --snapshot-names=<VM_1>-snap --zone=<ZONE_VM_1>
gcloud compute disks snapshot <VM_2> --snapshot-names=<VM_2>-snap --zone=<ZONE_VM_2>
gcloud compute snapshots list --filter="name~'<VM_PREFIX>-.*-snap'"
```

**Expected Output:**

| NAME | DISK_SIZE_GB | SRC_DISK | STATUS |
|------|---------------|-----------|--------|
| `<VM_1>-snap` | 24 | `<ZONE_VM_1>/disks/<VM_1>` | READY |
| `<VM_2>-snap` | 20 | `<ZONE_VM_2>/disks/<VM_2>` | READY |

---

## 2️⃣ Grant Access to the New Project

Find the Compute Engine default service account of the **new project**:

`<NEW_PROJECT_NUMBER>-compute@developer.gserviceaccount.com`

Add IAM permissions so that the new project can use these snapshots:

```bash
gcloud compute snapshots add-iam-policy-binding <VM_1>-snap   --member="serviceAccount:<NEW_PROJECT_NUMBER>-compute@developer.gserviceaccount.com"   --role="roles/compute.storageAdmin"

gcloud compute snapshots add-iam-policy-binding <VM_2>-snap   --member="serviceAccount:<NEW_PROJECT_NUMBER>-compute@developer.gserviceaccount.com"   --role="roles/compute.storageAdmin"

gcloud compute snapshots get-iam-policy <VM_1>-snap
```

---

## 3️⃣ Create Disks in the New Project

Switch context to the **new project**:

```bash
gcloud config set project <NEW_PROJECT_ID>
gcloud compute disks create <VM_1>-disk --source-snapshot=projects/<OLD_PROJECT_ID>/global/snapshots/<VM_1>-snap --zone=<ZONE_VM_1>
gcloud compute disks create <VM_2>-disk --source-snapshot=projects/<OLD_PROJECT_ID>/global/snapshots/<VM_2>-snap --zone=<ZONE_VM_2>
gcloud compute disks list
```

---

## 4️⃣ Recreate VMs in the New Project

Launch new instances using the created disks:

```bash
gcloud compute instances create <VM_1> --zone=<ZONE_VM_1> --machine-type=<VM_1_TYPE> --disk=name=<VM_1>-disk,boot=yes,auto-delete=no
gcloud compute instances create <VM_2> --zone=<ZONE_VM_2> --machine-type=<VM_2_TYPE> --disk=name=<VM_2>-disk,boot=yes,auto-delete=no
```

---

## 5️⃣ Post-Migration Tasks

### 🔹 Assign Static External IPs

```bash
gcloud compute addresses create <VM_1>-ip --region=<REGION>
gcloud compute instances add-access-config <VM_1> --access-config-name="External NAT" --address=<STATIC_IP>
```

### 🔹 Recreate Firewall Rules

```bash
gcloud compute firewall-rules create allow-http --allow=tcp:80
gcloud compute firewall-rules create allow-https --allow=tcp:443
```

### 🔹 Validate VM Access

```bash
gcloud compute ssh <VM_1> --zone=<ZONE_VM_1>
gcloud compute ssh <VM_2> --zone=<ZONE_VM_2>
```

---

## ✅ Migration Summary

✔️ Snapshots created in the old project.  
✔️ Access granted to the new project’s service account.  
✔️ Disks created in the new project from old snapshots.  
✔️ New VMs successfully launched from disks.  
✔️ Optionally reconfigured IPs and firewall rules.

---

## 🧾 Notes

- This method ensures **no downtime** during snapshot creation.  
- Snapshots are **incremental**, saving time and storage.  
- Always validate that **firewall rules** and **service accounts** are properly replicated.

---

**Author:** Blesslin  
**Date:** November 2025  
**Purpose:** GCP Migration Documentation 
