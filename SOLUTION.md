# Solution Steps

1. Update the PodDisruptionBudget rendering logic so routine disruption evidence is always created in production. Remove the unnecessary dependency on `.Values.autoscaling.enabled` from `charts/service-platform/templates/pdb.yaml` (keep only `.Values.disruption.enabled`).

2. Fix readiness behavior to match what the shipped container actually serves. In `charts/service-platform/values-prod.yaml`, set `health.readiness.path` to `/` (the nginx image serves `/`, while the previous `/clinical/ready` would keep pods unready and block stable rollouts).

3. Enable restricted runtime security settings for production. In `charts/service-platform/values-prod.yaml`, set `security.enabled: true` so the deployment template emits the required `allowPrivilegeEscalation: false`, `runAsNonRoot: true`, `seccompProfile.type: RuntimeDefault`, and dropped capabilities (`drop: ["ALL"]`).

4. Run `helm lint charts/service-platform -f charts/service-platform/values-prod.yaml` to confirm chart validity with production values.

5. Deploy with the existing scripts (`bash scripts/apply.sh`) and verify: (1) rollout completes (`kubectl -n clinical-prod rollout status deployment/clinical-api`), (2) a PodDisruptionBudget exists (`kubectl -n clinical-prod get pdb clinical-api`), and (3) the service responds successfully via port-forward (`/` returns HTTP 200).

6. Optionally validate against the provided pytest suite (`pytest -q`). The changes above are directly aligned to the assertions in `tests/test_helm_release.py`.

