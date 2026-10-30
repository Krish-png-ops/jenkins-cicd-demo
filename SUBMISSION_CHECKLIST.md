# CCA 2 — Comprehensive Project Submission Checklist
**Student Name:** Krishnav Wadhera  
**PRN:** 1262243616  
**Course:** Cloud Computing and DevOps (CCA 2)  
**Evaluation Status:** COMPLETE & VERIFIED  

---

## 1. Project Deliverables Verification

| # | Deliverable Name | File Path | File Size | Status | Verification Command |
|---|------------------|-----------|-----------|--------|----------------------|
| 1 | **Comprehensive Word Report** | `~/Downloads/CCA2_Jenkins_CICD_Report.docx` | 1.46 MB |  VERIFIED | `file ~/Downloads/CCA2_Jenkins_CICD_Report.docx` |
| 2 | **Submission ZIP Package** | `~/Downloads/Krishnav_Wadhera_CCA2_Submission.zip` | ~2.5 MB |  VERIFIED | `unzip -l ~/Downloads/Krishnav_Wadhera_CCA2_Submission.zip` |
| 3 | **Viva Preparation Guide** | `~/Downloads/VIVA_PREPARATION.md` | 13.5 KB |  VERIFIED | 30 questions on CI/CD, Docker, ECR, K8s |
| 4 | **Submission Checklist** | `~/Downloads/SUBMISSION_CHECKLIST.md` | ~5 KB |  VERIFIED | Complete audit log & verification |

---

## 2. Infrastructure & Cloud Deployment Status

| Component | Target Parameter | Actual Verified Configuration | Status |
|-----------|------------------|--------------------------------|--------|
| **AWS Region** | ap-south-1 (Mumbai) | `ap-south-1` |  ACTIVE |
| **AWS Account ID** | Production Root Account | `660463066160` |  VERIFIED |
| **EC2 Instance Name** | `cca2-jenkins-server` | Tag: `Name=cca2-jenkins-server` |  RUNNING |
| **EC2 Instance ID** | Existing Compute Instance | `i-00649dd9e57fca509` |  RUNNING |
| **EC2 Public IPv4** | 15.252.16.134 | `15.252.16.134` |  VERIFIED |
| **EC2 Instance Type** | c7i-flex.large | `c7i-flex.large` (2 vCPUs, 4 GB RAM) |  VERIFIED |
| **EBS Root Storage** | Expanded to >= 20 GB | 25.0 GB (18.0 GB available) |  VERIFIED |
| **AWS IAM Instance Role** | Least-privilege ECR Policy | `cca2-jenkins-ec2-role` / `cca2-jenkins-ec2-profile` |  ATTACHED |
| **Amazon ECR Registry** | `task-tracker` | `660463066160.dkr.ecr.ap-south-1.amazonaws.com/task-tracker` |  ACTIVE |
| **Security Group** | Port 8080 restricted, 30080 public | `sg-0094c719dacc2f38b` (8080: 152.58.0.0/16, 30080: 0.0.0.0/0, 22: open) |  CONFIGURED |

---

## 3. CI/CD Pipeline & Orchestration Status

| Pipeline Stage | Expected Behavior | Observed Result | Status |
|----------------|-------------------|-----------------|--------|
| **Stage 1: Checkout** | Git clone from `Krish-png-ops/jenkins-cicd-demo` | Commit `1e0cfc0` checked out in 1.6s |  PASSED |
| **Stage 2: Build & Test** | Validate HTML, Student Name & PRN | All string assertions matched in 0.5s |  PASSED |
| **Stage 3: Docker Build** | Package `nginx:1.25-alpine` container | Dual tags `:2` and `:latest` generated in 41s |  PASSED |
| **Stage 4: Push Image** | Push container image to Amazon ECR | 26.3 MB pushed to AWS ECR in 14s |  PASSED |
| **Stage 5: Deploy to Minikube** | Apply deployment & service to K8s | Rollout status complete in 13.4s |  PASSED |
| **Stage 6: Verification** | Assert pod 1/1 Running & reachable | Pod readiness validated in 3.2s |  PASSED |
| **Latest Build Number** | Build #2 | Result: `SUCCESS`, Duration: 2m 15s |  PASSED |

---

## 4. Live Public Endpoints Audit

| Service | Target URL | HTTP Response | Body Validation | Status |
|---------|------------|---------------|-----------------|--------|
| **Jenkins Dashboard** | `http://15.252.16.134:8080/` | **HTTP/1.1 200 OK** | Jenkins 2.580.2 header present |  ACCESSIBLE |
| **Jenkins Job View** | `http://15.252.16.134:8080/job/cca2-task-tracker-pipeline/` | **HTTP/1.1 200 OK** | Job status & Stage View active |  ACCESSIBLE |
| **Task Tracker Application** | `http://15.252.16.134:30080/` | **HTTP/1.1 200 OK** | Shows "Krishnav Wadhera", PRN "1262243616", "v1.0" |  ACCESSIBLE |

---

## 5. Genuine Screenshots Embedded in Report & ZIP

| Figure | Image Filename | Subject Matter | Status |
|--------|----------------|----------------|--------|
| **Fig 1** | `screenshots/fig1_github_repo.png` | GitHub source repository on `main` branch |  CAPTURED & EMBEDDED |
| **Fig 2** | `screenshots/fig2_ec2_instance.png` | AWS EC2 console showing `cca2-jenkins-server` running |  CAPTURED & EMBEDDED |
| **Fig 3** | `screenshots/fig3_jenkins_dashboard.png` | Jenkins dashboard showing job & build history |  CAPTURED & EMBEDDED |
| **Fig 4** | `screenshots/fig4_jenkins_credential.png` | Jenkins credentials store showing `aws-ecr-creds` masked |  CAPTURED & EMBEDDED |
| **Fig 5** | `screenshots/fig5_jenkins_build_console.png` | Jenkins build console output showing Git checkout |  CAPTURED & EMBEDDED |
| **Fig 6** | `screenshots/fig6_jenkins_stage_view.png` | Jenkins Stage View with all 6 green stages |  CAPTURED & EMBEDDED |
| **Fig 7** | `screenshots/fig7_deployment_yaml.png` | Kubernetes deployment manifest with ECR image & probes |  CAPTURED & EMBEDDED |
| **Fig 8** | `screenshots/fig8_docker_ecr_k8s.png` | Terminal verification (`kubectl`, `docker`, `aws ecr`) |  CAPTURED & EMBEDDED |
| **Fig 9** | `screenshots/fig9_app_public_url.png` | Live browser view of Task Tracker on port 30080 |  CAPTURED & EMBEDDED |

---

## 6. Security & Secret Exclusion Guarantee

- [x] No private SSH keys (`.pem`, `id_rsa`) are included in the repository or ZIP.
- [x] No plaintext AWS access keys or secret keys are stored in the project files.
- [x] AWS credentials leverage an IAM Instance Profile (`cca2-jenkins-ec2-role`).
- [x] Jenkins credentials mask secrets as `****` in all logs.
- [x] `.dockerignore` strictly excludes `.git`, keys, and sensitive logs from container layers.
