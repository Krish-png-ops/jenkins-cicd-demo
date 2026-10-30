# CCA 2 — Comprehensive Viva Voce Preparation Guide
**Student Name:** Krishnav Wadhera  
**PRN:** 1262243616  
**Subject:** Cloud Computing and DevOps (CCA 2)  
**Project:** End-to-End Automated Jenkins CI/CD Pipeline with Docker, Amazon ECR, and Kubernetes on AWS EC2  

---

## 30 Key Viva Voce Questions and Comprehensive Answers

### 1. What is CI (Continuous Integration)?
**Answer:** Continuous Integration (CI) is the software development practice where developers frequently commit code changes to a shared central repository (such as GitHub). Each commit triggers an automated pipeline that checks out the code, executes static code analysis, and runs automated build and test suites to detect bugs and integration issues immediately.

### 2. What is CD (Continuous Delivery vs. Continuous Deployment)?
**Answer:** Continuous Delivery automates the build, test, and release staging process, keeping the software deployable at any time, but requires manual approval for production rollout. Continuous Deployment automates the entire lifecycle through to production: every code change that passes all automated tests and validation gates is deployed directly to live infrastructure without manual intervention.

### 3. What is Jenkins and how does it function?
**Answer:** Jenkins is an open-source automation server designed to orchestrate CI/CD pipelines. It runs on Java (OpenJDK 17 in this project) as a daemon on an AWS EC2 instance. Jenkins monitors SCM repositories, provisions build workspaces, executes pipeline steps defined in a `Jenkinsfile`, integrates with third-party tools (Docker, AWS CLI, kubectl), and reports real-time execution telemetry.

### 4. What is a Jenkinsfile and what is the difference between Declarative and Scripted syntax?
**Answer:** A `Jenkinsfile` is a text file version-controlled in the Git repository that defines the CI/CD pipeline as code (Pipeline-as-Code). 
- **Declarative Pipeline:** Uses a strictly structured, opinionated syntax starting with `pipeline { agent any ... }`, offering cleaner readability, built-in stage views, and declarative post-actions. (Used in this project).
- **Scripted Pipeline:** Uses Groovy-based programming syntax (`node { ... }`), providing advanced algorithmic flexibility but with higher complexity.

### 5. What are the six pipeline stages implemented in this CCA 2 project?
**Answer:**
1. **Checkout:** Clones the GitHub repository `Krish-png-ops/jenkins-cicd-demo` on branch `main`.
2. **Build and Test:** Runs shell assertions verifying the presence of `index.html` and validating the student name (`Krishnav Wadhera`), PRN (`1262243616`), and version (`v1.0`).
3. **Docker Build:** Builds a container image using the project `Dockerfile` and tags it with both `${BUILD_NUMBER}` and `latest`.
4. **Push Image:** Authenticates against Amazon ECR via AWS IAM EC2 instance role credentials and pushes the tagged images to the private ECR repository.
5. **Deploy to Minikube:** Generates a dynamic Kubernetes `imagePullSecret` using the ECR auth token, applies `k8s/deployment.yaml` and `k8s/service.yaml`, and executes `kubectl rollout status`.
6. **Verification:** Inspects cluster health, asserts that pods are in the `Running` state, and validates external reachability on NodePort `30080`.

### 6. Why is containerization with Docker used in this project?
**Answer:** Docker packages the Task Tracker application, its static web assets, and the lightweight Nginx web server into an immutable, portable container image (`nginx:1.25-alpine`). This eliminates the "it works on my machine" problem, standardizes deployment environments, isolates runtime dependencies, and enables microservice scaling.

### 7. What is Amazon ECR and why is it preferred over public Docker Hub?
**Answer:** Amazon Elastic Container Registry (ECR) is a managed AWS container registry offering high availability, server-side encryption (KMS), and native IAM integration. It is preferred because it integrates seamlessly with AWS EC2 instance profiles, avoiding hardcoded static passwords, eliminating Docker Hub rate limits, and securing private enterprise container images within the AWS network boundary.

### 8. How does the pipeline authenticate to Amazon ECR securely without hardcoding AWS keys?
**Answer:** The EC2 compute instance (`cca2-jenkins-server`) is attached to an AWS IAM Instance Profile (`cca2-jenkins-ec2-profile`) bound to IAM Role `cca2-jenkins-ec2-role` with the `AmazonEC2ContainerRegistryPowerUser` managed policy. When Jenkins executes `aws ecr get-login-password --region ap-south-1`, the AWS CLI automatically retrieves temporary STS security credentials from the EC2 Instance Metadata Service (IMDS). In addition, Jenkins masks the credential value as `aws-ecr-creds`.

### 9. What is Kubernetes and what role does it play in this architecture?
**Answer:** Kubernetes is a container orchestration platform that automates container scheduling, scaling, networking, and lifecycle management. In this project, a Minikube cluster (v1.39.0 / Kubernetes v1.31.0) manages the `task-tracker` Deployment, ensuring desired state reconciliation, automated container restarts, and zero-downtime rolling updates.

### 10. Explain the role of `k8s/deployment.yaml`.
**Answer:** `k8s/deployment.yaml` declaratively defines the desired state for running the application:
- **Replicas:** Specifies `replicas: 1`.
- **Image:** Specifies the private Amazon ECR URI: `660463066160.dkr.ecr.ap-south-1.amazonaws.com/task-tracker:latest`.
- **ImagePullSecrets:** References `ecr-secret` so the Kubernetes kubelet can authenticate against private ECR to pull images.
- **Readiness Probe:** Probes HTTP GET `/` on port 80 (initial delay 5s) to ensure traffic is only routed when Nginx is healthy.
- **Liveness Probe:** Probes HTTP GET `/` on port 80 (initial delay 15s) to restart the container if the web server hangs.

### 11. Explain the role of `k8s/service.yaml` and the NodePort concept.
**Answer:** A Kubernetes Service provides a stable network endpoint for a group of pods. A `NodePort` service exposes the application on a static port across all cluster nodes (in our configuration, port `30080`). Traffic directed to `NodePort 30080` is automatically proxied to port 80 of the running `task-tracker` pod.

### 12. How is traffic bridged from the EC2 host's public IP to the Minikube cluster?
**Answer:** Minikube runs inside a Docker container on the EC2 host on a virtual network bridge (`192.168.49.2`). Because external traffic hits the host's public network interface, a persistent `socat` background daemon (`nodeport-proxy.service`) forwards incoming connections from host TCP port 30080 directly to `192.168.49.2:30080`.

### 13. What happens if a developer pushes code where the PRN or student name is missing?
**Answer:** During Stage 2 (`Build and Test`), the automated pipeline executes shell assertions:
```bash
grep -q "Krishnav Wadhera" index.html || { echo "Missing Student Name"; exit 1; }
grep -q "1262243616" index.html || { echo "Missing PRN"; exit 1; }
```
If either assertion fails, the command returns exit code 1. Jenkins immediately halts the pipeline, marks the build as `FAILED` (red), aborts container building and deployment, and logs the specific error.

### 14. What is the significance of `.dockerignore`?
**Answer:** `.dockerignore` specifies files and directories that Docker CLI must exclude when generating the build context. In our project, it prevents the `.git/` history directory, `.pem` SSH keys, local markdown notes, and temporary build files from being sent to the Docker daemon, resulting in faster image builds and preventing sensitive credential leakage.

### 15. What are Kubernetes Readiness and Liveness Probes and why are both necessary?
**Answer:**
- **Readiness Probe:** Determines whether a pod is ready to accept user traffic. If the probe fails, the pod's IP is removed from the service endpoints so users do not experience connection errors during container startup.
- **Liveness Probe:** Determines whether the application container is still alive. If the web server deadlocks or crashes, kubelet terminates the container and creates a replacement pod based on the restart policy.

### 16. How did you handle AWS Security Group configurations for Jenkins and the application?
**Answer:** The security group (`sg-0094c719dacc2f38b`) was hardened using defense-in-depth principles:
- **Port 8080 (Jenkins):** Restricted to the user's carrier CIDR block (`152.58.0.0/16`) to protect the CI/CD orchestrator from arbitrary unauthorized access.
- **Port 30080 (Application):** Opened globally (`0.0.0.0/0`) so instructors and external evaluators can inspect the running application.
- **Port 22 (SSH):** Preserved for encrypted administrative access.

### 17. How is image versioning handled in the pipeline?
**Answer:** Every build generates two tags:
- `${BUILD_NUMBER}` (e.g., `:2`): An immutable, unique tag matching the Jenkins build run, enabling immediate rollback to any historical version.
- `latest`: A mutable convenience tag representing the newest verified build in production.

### 18. What is the purpose of `post` actions in a Jenkins pipeline?
**Answer:** The `post` section executes conditional logic after all stages finish. `post { always { ... } success { ... } failure { ... } }` is used to send build notifications, clean up temporary workspace files, or output deployment confirmation links.

### 19. What disk and memory challenges did you encounter on the EC2 instance, and how were they resolved?
**Answer:**
1. **EBS Disk Limit:** The default 8.0 GiB root volume was exhausted by Minikube base images and Docker layers. The volume was modified to 25.0 GiB via AWS CLI and expanded live using `growpart` and `xfs_growfs`.
2. **Tmpfs Threshold:** Jenkins' `DiskSpaceMonitor` flagged `/tmp` because the default tmpfs mount was 457 MB. It was remounted with `size=4G`.
3. **RAM Constraints:** A 4.0 GiB swap file was configured to support Docker builds and Minikube containerd runtime simultaneously.

### 20. What is Git SCM polling vs. Webhooks?
**Answer:**
- **Polling SCM:** Jenkins periodically connects to GitHub to check if new commits exist.
- **Webhooks:** GitHub actively sends an HTTP POST payload to the Jenkins `/github-webhook/` endpoint the instant a push event occurs, triggering immediate pipeline execution with zero latency.

### 21. What is an immutable artifact in CI/CD?
**Answer:** An immutable artifact is a compiled container image or binary package that is never modified after it is built. In this project, once Docker builds and pushes `task-tracker:2` to Amazon ECR, that image is never changed; any bug fix or update requires building a new tag (`:3`).

### 22. What command allows you to inspect live Kubernetes pod logs?
**Answer:** `kubectl logs -l app=task-tracker -f` streams live logs from the Nginx container inside the pod.

### 23. What command checks the status of a rolling update in Kubernetes?
**Answer:** `kubectl rollout status deployment/task-tracker` tracks whether new replica pods have successfully reached the ready state and replaced older pods.

### 24. What is the difference between ClusterIP and NodePort services?
**Answer:** `ClusterIP` provides an internal virtual IP reachable only from within the Kubernetes cluster. `NodePort` exposes the service on an allocated high-range port (30000–32767) across every cluster node's external IP, enabling external ingress without an enterprise Cloud Load Balancer.

### 25. Why is Nginx chosen to serve the HTML web page in Docker?
**Answer:** Nginx is an event-driven, asynchronous web server capable of handling thousands of concurrent HTTP connections with negligible memory usage (< 15 MB), providing sub-millisecond response times for static HTML/CSS assets.

### 26. How do you verify the Jenkins pipeline build status programmatically?
**Answer:** Using the Jenkins REST API:
```bash
curl -s -u admin:AdminPassword123! http://15.252.16.134:8080/job/cca2-task-tracker-pipeline/2/api/json
```
This returns a JSON payload including `result: "SUCCESS"` and duration timestamps.

### 27. What is least-privilege access and how was it demonstrated?
**Answer:** The Principle of Least Privilege dictates that an entity should only possess the minimum permissions necessary to perform its intended function. Instead of assigning `AdministratorAccess` to the EC2 instance, the IAM role was granted solely `AmazonEC2ContainerRegistryPowerUser` to interact with Amazon ECR.

### 28. What is the role of `kubectl apply` vs `kubectl create`?
**Answer:** `kubectl create` is an imperative command that fails if the resource already exists. `kubectl apply` is declarative: it inspects the live cluster state, calculates the delta between the YAML file and the running object, and applies updates in-place without downtime.

### 29. How would you perform an instant rollback if a production bug is discovered?
**Answer:** Using Kubernetes native rollback commands:
```bash
kubectl rollout undo deployment/task-tracker
```
This instructs Kubernetes to immediately revert the pod template to the previous revision tag.

### 30. Summarize the end-to-end flow of this project in one sentence.
**Answer:** A developer commits web code to GitHub, triggering a 6-stage Jenkins pipeline on AWS EC2 that validates student metadata, builds and pushes an immutable Docker image to Amazon ECR via IAM instance profile credentials, and deploys it to a Kubernetes Minikube cluster with zero downtime, verified live on public NodePort 30080.
