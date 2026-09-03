Verification, troubleshooting, and key lessons
Verification commands used
Check ArgoCD services and pods:

bash
kubectl get pods -n argocd
kubectl get svc -n argocd
Check application and resources:

bash
kubectl get applications -n argocd
kubectl get pods -n default
kubectl describe deployment <app> -n default
Confirm Docker image:

bash
docker pull <dockerhub-user>/<image>:<tag>
Common issues encountered and fixes
minikube service not finding service — caused by namespace mismatch. Fix:

bash
minikube service argocd-server -n argocd
Service type reverted to ClusterIP — operator reconciliation overwrote manual edits. Fix: update the ArgoCD CR spec and reapply.

Invalid admin login — decoded password from the wrong secret/field. Fix: list secrets and decode the correct secret field (see commands above) or reset the admin password via patching the argocd-secret.

Base64 decode errors — use base64 -d or base64 --decode.

Jenkins Git push errors — ensured Jenkins had proper Git credentials and workspace ownership fixed via git config --global --add safe.directory <workspace>.

Key takeaways
Operator pattern: CRD defines resource types; CR instances drive actual deployments. Always change operator-managed behavior via the CR.

Namespaces matter: Always specify -n <namespace> or set the kubectl context namespace.

GitOps principle: Git is the single source of truth; ArgoCD continuously reconciles cluster state to match Git.

Secrets: Kubernetes secrets are base64-encoded; decode with kubectl + base64 -d to avoid copy/paste errors.

Automate safely: Use immutable image tags (build number or SHA) and rotate credentials after first use.
