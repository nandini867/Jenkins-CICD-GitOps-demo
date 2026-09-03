cat > LEARNING_NOTES.md <<'EOF'
# LEARNING_NOTES

## Summary
This file captures concise learning points, the key errors encountered during the project, and the exact troubleshooting steps that resolved them. Use this as a quick reference when reproducing the CI/CD pipeline (Jenkins → SonarQube → DockerHub → ArgoCD → Kubernetes).

---

## Key Learning Points
- **Operator pattern matters**  
  Operators reconcile cluster state from a Custom Resource. Do not edit operator-owned resources directly; change the operator CR and reapply.

- **Namespaces are critical**  
  `kubectl` and many tools default to the `default` namespace. Always use `-n <namespace>` or set the context namespace when working with ArgoCD or operator resources.

- **Secrets are base64 encoded**  
  Always decode secrets using `kubectl` piped to `base64 -d` to avoid copy/paste errors and to ensure you read the correct field.

- **Git is the single source of truth**  
  Make manifest changes in Git. ArgoCD continuously reconciles the cluster to match Git; manual edits will be reverted.

- **Immutable image tags**  
  Use build numbers or SHA digests for image tags in manifests to ensure reproducible deployments and clear traceability.

- **Automate safely**  
  Use ArgoCD `syncPolicy.automated` with `prune` and `selfHeal` only after validating manifests and health checks in staging.

---

## Errors Faced and Root Causes
- **minikube service SVC_NOT_FOUND**  
  *Root cause:* Command searched `default` namespace.  
  *Symptom:* `Service 'argocd-server' was not found in 'default' namespace.`

- **Service type reverted to ClusterIP**  
  *Root cause:* Manual edit of operator-owned Service; operator reconciled it back.  
  *Symptom:* NodePort change did not persist.

- **Invalid username or password for ArgoCD admin**  
  *Root cause:* Decoded value came from the wrong secret or wrong field (`example-argocd-cluster.admin.password` vs the actual `argocd-initial-admin-secret.password`).  
  *Symptom:* Login shows "Invalid username or password".

- **base64 decode errors on CLI**  
  *Root cause:* Wrong flag usage (`decode` or `-decode` instead of `-d` or `--decode`).  
  *Symptom:* `base64: invalid option -- 'e'` or `decode: No such file or directory`.

- **Jenkins Git push failures from pipeline**  
  *Root cause:* Jenkins workspace ownership or missing Git credentials.  
  *Symptom:* `fatal: unsafe repository` or authentication errors when pushing.


## Troubleshooting Steps and Exact Commands
### Namespace and service discovery
- List services in argocd namespace:
```bash
kubectl get svc -n argocd


Open ArgoCD service via Minikube:
minikube service argocd-server -n argocd

**Decode the correct admin password**
If secret field is `password`:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo


Fix service type reversion
Edit the ArgoCD CR spec to set service type to NodePort and reapply:
kubectl apply -f example-argocd.yaml -n argocd
Do not edit the Service directly.

Jenkins workspace ownership fix
# On Jenkins host
git config --global --add safe.directory /var/lib/jenkins/workspace/<job-name>
Ensure Jenkins has Git credentials and push permissions.

Short Best Practices
Edit CRs not operator resources.

Always specify namespace.

Use kubectl to fetch and decode secrets.

Keep Git as the single source of truth.

Use immutable tags and rotate credentials.
