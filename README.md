# Self-Hosted GitHub Actions Runner — Demo
Demo project for setting up a self-hosted GitHub Actions runner

**Repository:** `<(https://github.com/Taiwo-Peter2023/self-hosted-runner-demo)>`  
**Owner:** `<Taiwo-Peter2023>`  
**Created:** `<22/10/2025>`

---

## Overview

This repository demonstrates how to install, configure, and validate a **self-hosted GitHub Actions runner** on a Windows machine (local PC). It includes:

- Step-by-step installation instructions
- A test GitHub Actions workflow (`.github/workflows/test.yml`)
- Troubleshooting and fixes implemented during setup
- Security considerations and production recommendations

---

## Repository structure
/ (repo root)
├─ README.md
├─ .github/
│ └─ workflows/
│ └─ test.yml
└─ actions-runner/ # recommended local folder (not committed)

---


## Prerequisites

- GitHub account and repository with Admin access.
- Windows PC with administrative rights.
- Internet access to `api.github.com`.
- PowerShell (run as Administrator).
- Git (for local commits, optional).

---

## Local runner install & configuration — Commands used

> Create a folder recommended by GitHub (example: `C:\actions-runner`) and run PowerShell as Administrator.

```powershell
# Create folder and move into it
mkdir C:\actions-runner
cd C:\actions-runner

# Download latest runner package (replace version if newer)
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.320.0/actions-runner-win-x64-2.320.0.zip -OutFile actions-runner.zip

# Extract
Add-Type -AssemblyName System.IO.Compression.FileSystem
[IO.Compression.ZipFile]::ExtractToDirectory("actions-runner.zip", "$PWD")

# Configure runner (get token from GitHub: Settings → Actions → Runners → New self-hosted runner)
.\config.cmd --url https://github.com/<your-username>/<your-repo> --token <REGISTRATION_TOKEN>

# Start runner (keep this PowerShell window open)
.\run.cmd

# Optional: install as Windows Service to auto-start on reboot
.\svc install
.\svc start


name: Test Self-Hosted Runner

on:
  push:
    branches:
      - main

jobs:
  test-job:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Display runner information
        run: |
          echo "✅ This workflow is running on a self-hosted runner!"
          echo "Runner Machine Name:"
          hostname
          echo "Current Directory:"
          pwd
          echo "Listing Files:"
          dir

      - name: Run test command
        run: echo "🎉 Self-hosted runner executed successfully!"

How to validate

Ensure the runner process (.\run.cmd) is running on your machine and shows Listening for Jobs.

Commit and push the workflow file to main (or your default branch).

Open the Actions tab → click the Test Self-Hosted Runner workflow run.

If the runner picked the job up, the logs will show the hostname and the confirmation message.

In GitHub: Settings → Actions → Runners you should see the runner listed with Online or Busy (while executing).

Troubleshooting (errors encountered & fixes)
1. 'Invoke-WebRequest' is not recognized

Cause: Running the PowerShell command in Command Prompt.
Fix: Open PowerShell (or run powershell inside cmd), then run the Invoke-WebRequest command.

2. 404 Not Found on registration (POST /actions/runner-registration)

Cause: Wrong URL, expired token, wrong scope (org vs repo), or lack of admin permission.
Fixes:

Use correct --url format: https://github.com/<username>/<repo>.

Generate a new registration token immediately before running config.cmd.

Ensure you have Admin access, or generate token from the organization settings if repo is under org.

Try from an unrestricted network if company firewall blocks api.github.com.

3. Trying to type self-hosted or Online in PowerShell

Cause: These are labels/status on GitHub, not commands.
Fix: Run .\run.cmd to start the runner. Leave the window open.

4. running scripts is disabled on this system (PSSecurityException)

Cause: PowerShell execution policy prevents running temporary scripts that Actions creates.
Fix:

# Recommended (persistent on machine)
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine
# Or temporary for the session:
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

Challenges I faced & how I solved them

PowerShell vs Command Prompt confusion

Symptom: Invoke-WebRequest not recognized.

Solution: Run commands in PowerShell (PS prompt) rather than cmd.exe.

Short-lived registration token / 404 error

Symptom: 404 from POST /actions/runner-registration.

Solution: Regenerate token and re-run config.cmd; verify correct repo URL and admin access.

Execution policy blocking scripts

Symptom: running scripts is disabled on this system error.

Solution: Set execution policy to RemoteSigned or set the Process scope to Bypass during setup.

Firewall / company network blocking GitHub API

Symptom: Cannot reach api.github.com.

Solution: Use a different network (home Wi-Fi or mobile hotspot) or work with IT to allow outbound GitHub traffic.

Production recommendations — what I would do differently

Dedicated runner hosts: Use VMs, containers (Kubernetes), or cloud VMs instead of personal machines.

Ephemeral runners: Prefer ephemeral runners (spawn-per-job) to limit stateful contamination and reduce security exposure.

Auto-scaling & orchestration: Use autoscaling groups or Kubernetes runners to scale with load.

Monitoring & alerting: Integrate with monitoring (Prometheus/Grafana) and log aggregation (ELK, Datadog).

Least-privilege tokens: Use tokens and credentials with minimal permissions and rotate them regularly.

Network isolation: Place runners in VPC/subnet with restricted egress and only allow required endpoints.

Immutable images: Deploy runners from trusted, immutable images to ensure consistent environment.

Security considerations implemented

Execution policy: Set to RemoteSigned to allow safe local scripts and block unsigned remote scripts.

No secrets in code: All secrets stored in GitHub Secrets, not in plaintext in workflows.

Limited token use: Registration token used only for config; removed from disk.

Access control: Only repository admins can register runners.

Service account / dedicated user: In production, run runner under a dedicated service account with minimal OS permissions.

Firewall rules & network segmentation: Limit which IPs/hosts runner can reach where practical.




![Uploading Test Job_Success .jpg…]()
