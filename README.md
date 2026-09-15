# Warehouse System Version Upgrade Deployment SOP


## Table of Contents

1. [Pre-Upgrade Checklist](#1-pre-upgrade-checklist)
2. [Upgrade Steps](#2-upgrade-steps)
3. [Rollback Plan](#3-rollback-plan)
4. [Post-Upgrade Verification & Testing](#4-post-upgrade-verification--testing)

---

## 1. Pre-Upgrade Checklist

Confirm each item before starting any deployment operation.

### 1.1 Permissions & Access

- [ ] Confirmed access to the GitHub repository (`rr_oks_deploy`)
- [ ] SSH Key configured and able to connect to GitHub

### 1.2 Version & Configuration Confirmation

- [ ] Target deployment version confirmed
- [ ] All fields in `values.source.yaml` verified as correct
- [ ] Site-specific configurations reviewed (e.g., `signal_tower_config.yaml`, `skeleton_tracking`, `no_robots`, etc.)
- [ ] Pull Request submitted to `rr_oks_deploy` with OKS Systems team members assigned as reviewers
- [ ] Pull Request approved by the OKS Systems team before proceeding

### 1.3 On-Site Status Confirmation

- [ ] **All robots returned to Home Tile**
- [ ] **Global mode switched to E-Stop**
- [ ] Confirmed no tasks or transport operations currently in progress
- [ ] On-site manager notified that upgrade is about to begin

### 1.4 Data Backup

- [ ] Execute database backup (~10 minutes):

  ```bash
  just upload-db prod/<SITE_NAME>
  ```

  Example: `just upload-db prod/pco-hikone-001`

- [ ] Confirmed backup completed successfully with no errors

---

## 2. Upgrade Steps

### Step 1: Execute GitHub Actions Deployment

1. Go to the GitHub Actions page:  
   `https://github.com/rapyuta-robotics/rr_oks_deploy/actions/workflows/deploy.yml`

2. Click **"Run Workflow"** and fill in the following parameters:

   | Parameter | Value |
   |---|---|
   | Environment | `prod` |
   | Target site path | `prod/<SITE_NAME>` (e.g., `prod/nx-daiba-001`) |
   | rr_oks_deploy branch | Your branch name (e.g., `pco-hikone-001`) |
   | Component | `all` (full deployment) or specify a component |
   | Upload configs | ✅ Check if configuration changes are included |
   | Force deploy | ✅ Check if required |

3. Click **"Run workflow"** to start deployment (estimated ~60 minutes).

4. Monitor the run status on the Actions page:
   - ✅ Green: Deployment successful
   - ❌ Red: Deployment failed — contact #oks_systems

---

## 3. Rollback Plan

Execute the following steps if the system is abnormal after upgrade.

### 3.1 Immediate Response

1. Notify the on-site manager to suspend all robot operations
2. Ensure all robots are in E-Stop state

### 3.2 Rollback Execution

**Option A: Redeploy using the main branch (Recommended)**

1. Trigger GitHub Actions with the same parameters as [Step 1](#step-1-execute-github-actions-deployment), but set `rr_oks_deploy branch` to `main`

**Option B: Restore database backup (for data anomalies)**

1. Contact the OKS Systems team for the database restore command
2. Execute the restore and confirm data integrity

### 3.3 Post-Rollback Confirmation

- [ ] GHA rollback deployment completed successfully
- [ ] System status returned to normal
- [ ] All robots communicating normally

---

## 4. Post-Upgrade Verification & Testing

Complete the following verification within approximately **3 hours** after deployment (~180 minutes).

### 4.1 System-Level Verification

- [ ] GHA run result shows success (✅)
- [ ] System management UI shows all services normal (no alerts)
- [ ] Version number updated to target version

### 4.2 Robot Basic Function Verification

- [ ] All robots online and communicating with the system
- [ ] Robots can recover from E-Stop and accept dispatch instructions
- [ ] Robot navigation and localization functioning normally

### 4.3 Business Function Verification

- [ ] Manually trigger a test task and confirm the full flow from order to robot execution
- [ ] Confirm GWM (Global Warehouse Manager) task scheduling is working
- [ ] Confirm UI displays correctly with no errors

### 4.4 Completion Report

- [ ] Post upgrade completion notice in Slack #oks_systems, including:
  - Site name
  - Deployed version
  - Completion time
  - Any outstanding issues
- [ ] Update relevant deployment records

---

## Appendix

### Quick Command Reference

| Operation | Command |
|---|---|
| Database backup | `just upload-db prod/<SITE_NAME>` |
| Create branch | `git checkout -b <SITE_NAME>` |
| Commit changes | `git commit -m "deploy: <SITE_NAME> to <VERSION>"` |
| Push branch | `git push origin <SITE_NAME>` |

### Site Path Formats

| Type | Format | Example |
|---|---|---|
| Config file path | `sites/prod/<SITE_NAME>/values.source.yaml` | `sites/prod/nx-daiba-001/values.source.yaml` |
| GHA Target site path | `prod/<SITE_NAME>` | `prod/pco-hikone-001` |

### When Issues Arise

Contact Slack channel: **#oks_systems**  
Reference PR: `https://github.com/rapyuta-robotics/rr_oks_deploy/pull/2274`
