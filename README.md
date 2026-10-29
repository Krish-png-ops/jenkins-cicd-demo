# CCA 2 — CI/CD Pipeline Implementation

## Student details
- **Name:** Krishnav Wadhera
- **PRN:** 1262243616
- **Course:** Cloud Computing and DevOps

## Project overview
This minimal project demonstrates a Jenkins declarative pipeline that checks out the repository, verifies that `index.html` exists and is non-empty, and checks that the page includes the expected assignment details. It does not deploy to a server or create AWS resources.

## Project structure
```text
.
├── index.html   # Styled demo page with student details and version v1.0
├── Jenkinsfile  # Three-stage Jenkins declarative pipeline
└── README.md    # Project documentation
```

## Prerequisites
- A GitHub repository containing these files
- Jenkins with Pipeline support
- A Jenkins Pipeline job configured to use this repository and its `Jenkinsfile`

## Run the pipeline in Jenkins
1. Push the project files to the repository's `main` branch.
2. In Jenkins, select **New Item** and create a **Pipeline** job (or open an existing one).
3. Under **Pipeline**, select **Pipeline script from SCM**.
4. Select **Git**, enter the repository URL, select the `main` branch, and set the script path to `Jenkinsfile`.
5. Save and select **Build Now**.
6. Open **Console Output** or **Stage View**. The expected stages are **Checkout**, **Validate File**, and **Test & Quality Check**.

## Checks performed
- `index.html` exists.
- `index.html` is not empty.
- The page contains `CI/CD Pipeline Demo`, `Krishnav Wadhera`, `1262243616`, and `v1.0`.

## Scope
This is a basic validation pipeline for the assignment. It does not install Jenkins, provision EC2, or deploy the page to a web server.
