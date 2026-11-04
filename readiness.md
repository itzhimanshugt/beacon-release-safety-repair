# Beacon release review

## Pipeline audit
Gap / risk / repaired dependency: Production job had no needs, so it could run even if validation failed; repaired with needs: validate.
Production reviewer control to configure (not demonstrated locally): Required reviewers must be set in GitHub Settings > Environments > production; the environment name alone does not enforce approval.
Secret handling and scope: Store REGISTRY_TOKEN as an environment secret scoped to production only; never print or echo it in logs.

## Traceability
Why PATCH is appropriate: It is a backward-compatible bug fix with no new features or breaking changes, so only the PATCH number rises (v2.3.0 to v2.3.1).
Candidate image and annotated Git tag: Image beacon-service:v2.3.1 with annotated Git tag v2.3.1 (the tag points to the code commit and does not prove image provenance).
Record tag -> code commit in PDF using git rev-parse commands.
Image-tag mutability limitation: A version tag like v2.3.1 can be re-pushed to point at a different image, while a verified digest (sha256) cannot.

## Incident recovery
Evidence and rollback vs forward decision: Supplied fixture, not observed: 35% of requests fail, errors rising, 2 of 3 readiness checks fail, root cause unknown. Decision: roll back, because v2.3.0 is verified healthy and there is no migration.
Known-good target: beacon-service:v2.3.0
Proposed kubectl command (not executed): kubectl set image deployment/beacon beacon=beacon-service:v2.3.0
Recovery checks after action: kubectl rollout status deployment/beacon completes, all 3 readiness checks pass, and the request failure rate returns to normal.

## Release note and decision
Change scope: Three config fixes only (pipeline needs, image tag v2.3.1, rollback target v2.3.0); no application code changed.
Local validation proof: python3 check_release.py all shows pipeline, image and rollback PASS (static contract checks only).
Open risk / owner / mitigation: Image tag is mutable; owner is the release engineer; mitigation is to pin and verify an image digest before deploying.
Current production decision and blocker: NO-GO. Approval is NOT REQUESTED, the candidate is NOT DEPLOYED, and required reviewers and the image digest are unverified.
Terraform vs Kubernetes responsibilities: Terraform provisions the infrastructure (clusters, networks), while Kubernetes runs and operates the workloads on it.
