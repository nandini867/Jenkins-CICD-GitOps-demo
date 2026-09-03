Overview
This document records reproducible, end‑to‑end steps performed for the project: full CI (Jenkins → SonarQube → Docker → DockerHub) and full CD (ArgoCD Operator → ArgoCD Application → Kubernetes/Minikube). It is intended to be committed to your repository as the canonical “steps done” reference. This file intentionally does not include full YAML manifests or the Jenkinsfile contents — those live in the repo under k8s/ and the repo root respectively.

Prerequisites
Local tools: kubectl, minikube (for local testing), docker, git.

CI host: Jenkins server (EC2 or VM) with Docker installed and Jenkins user able to run Docker.

Accounts: DockerHub account; GitHub repo containing application source and Kubernetes manifests.

Cluster: Minikube or any Kubernetes cluster with sufficient resources for ArgoCD and the app.

Access: SSH/console access to Jenkins host and kubectl configured to the target cluster.

CI — Jenkins, SonarQube, Docker, and Pipeline (what was done)
Jenkins installation and configuration
Installed Jenkins on a host (EC2/VM), enabled and started the service.

Installed required plugins via Jenkins UI: Pipeline, Git, Docker Pipeline, SonarQube Scanner, Credentials, GitHub integration.

Created credentials in Jenkins for DockerHub, GitHub (token), and SonarQube (token).

Configured Jenkins agents (if used) and ensured the Jenkins user can run Docker commands (or used Docker-in-Docker where appropriate).

SonarQube setup and integration
Installed SonarQube on a server; adjusted kernel and memory settings (vm.max_map_count) as required.

Created a Sonar token and added it to Jenkins credentials.

Integrated SonarQube into the Jenkins pipeline so the pipeline runs static analysis and honors the Sonar quality gate.

Docker and DockerHub
Built Docker images in the Jenkins pipeline and pushed images to DockerHub under the project account.

Verified images by pulling from DockerHub on a test host.

Pipeline behavior (high level)
Checkout source from GitHub.

Build & test the application (Maven/Gradle).

Run SonarQube analysis and fail the build if quality gate fails.

Build Docker image and push to DockerHub with a unique tag (build number or SHA).

Update Kubernetes manifests in the Git repo (image tag) and push the commit so ArgoCD can pick it up.

Fixed common CI issues encountered during setup
(example: Git “dubious ownership” on Jenkins workspace by adding the workspace to safe.directory).

CD — ArgoCD Operator, ArgoCD access, Application and GitOps flow (what was done)
Install ArgoCD Operator and create ArgoCD instance
Installed the ArgoCD Operator (Operator namespace + operator deployment) so ArgoCD components are managed via a single ArgoCD Custom Resource (CR).

Created the ArgoCD namespace and applied an ArgoCD CR to instruct the operator how to configure ArgoCD (service type, TLS, resource settings).

Waited for reconciliation and verified ArgoCD components (pods, services, CRDs) were created by the operator.

Important: operator-managed resources are reconciled automatically. All changes to service type, TLS, or operator-managed settings were made by editing the ArgoCD CR and reapplying it — manual edits to operator-owned Services/Deployments were avoided.


Creating an ArgoCD Application and GitOps flow
Created an Application resource in the argocd namespace that points to the GitHub repository and the path containing Kubernetes manifests (or Helm chart).

Enabled automated sync (optional) with prune: true and selfHeal: true so ArgoCD automatically applies manifest changes pushed to Git.

Verified that when Jenkins updated the manifest image tag and pushed to Git, ArgoCD detected the change and synced the cluster, resulting in updated pods pulling the new image.
