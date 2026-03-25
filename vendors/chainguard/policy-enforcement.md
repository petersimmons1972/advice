# Chainguard: Policy Enforcement in Kubernetes

**Source:** Chainguard Academy, Container Security Best Practices
**Last updated:** 2026-03-25
**Scope:** Enforcing CHPs (Container Hardening Priorities) at deployment time and runtime

## Overview

Policy enforcement means automatically preventing misconfigured or insecure deployments from running. Instead of hoping developers follow guidelines, policies **reject** violations at admission time.

**Philosophy:** Make secure defaults easy, insecure configurations hard.

**Without policy enforcement:**
- ❌ Developer deploys container as root → Works, but vulnerable
- ❌ Privileged container slips through → Runs, can escape
- ❌ Unscanned image deployed → No detection

**With policy enforcement:**
- ✅ Root deployment rejected at admission → Must use non-root
- ✅ Privileged container blocked → Must drop capabilities
- ✅ Unscanned image blocked → Must pass vulnerability gate

## Two Policy Engines for Kubernetes

### Option 1: Kyverno (Recommended for CHPs)

**What:** Kubernetes-native policy engine, simple YAML syntax
**Best for:** Container security policies (images, capabilities, privileges)
**Complexity:** Low, uses familiar Kubernetes YAML

**Example: Require non-root user**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  rules:
  - name: check-runAsNonRoot
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Running as root is not allowed"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
```

**Deployment that FAILS:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: root-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    # No securityContext → runs as root → REJECTED
```

**Deployment that PASSES:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: safe-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 65532  # Non-root UID
  containers:
  - name: app
    image: myapp:latest
```

### Option 2: OPA/Gatekeeper (Powerful but Complex)

**What:** General-purpose policy engine, uses Rego language
**Best for:** Complex policies, multi-cloud policies, non-Kubernetes policies
**Complexity:** High, requires Rego programming

**Example: Same non-root policy in Rego**
```rego
package kubernetes.admission

deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    not container.securityContext.runAsNonRoot
    msg := sprintf("Container %v must have runAsNonRoot: true", [container.name])
}
```

**Trade-off:**
- Kyverno: Simpler, faster to implement, covers 80% of use cases
- OPA: More powerful, handles complex logic, steeper learning curve

**Recommendation for Chainguard + CHPs:** Start with Kyverno.

## CHP-to-Policy Mapping

Each Container Hardening Priority pillar has corresponding policies:

### Pillar 1: Minimalism → Image & Runtime Policies

**Policy: Distroless/minimal base images only**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-distroless
spec:
  validationFailureAction: enforce
  rules:
  - name: check-distroless
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Image must be from distroless registry (chainguard, google)"
      pattern:
        spec:
          containers:
          - image: "cgr.dev/chainguard/* | gcr.io/distroless/*"
```

**Policy: No privileged containers**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-privileged
spec:
  validationFailureAction: enforce
  rules:
  - name: privileged-container
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Privileged containers are not allowed"
      pattern:
        spec:
          containers:
          - securityContext:
              privileged: false
```

**Policy: Drop all capabilities, add back only needed**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: drop-all-caps
spec:
  validationFailureAction: audit  # Start with audit, enforce later
  rules:
  - name: drop-caps
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Must drop ALL capabilities"
      pattern:
        spec:
          containers:
          - securityContext:
              capabilities:
                drop:
                - ALL
```

**Policy: Read-only root filesystem**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-ro-root
spec:
  validationFailureAction: audit  # Audit first, enforce later
  rules:
  - name: read-only-root
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Root filesystem must be read-only"
      pattern:
        spec:
          containers:
          - securityContext:
              readOnlyRootFilesystem: true
```

### Pillar 2: Provenance → Image Verification

**Policy: Only signed images allowed**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-image-signature
spec:
  validationFailureAction: enforce
  rules:
  - name: check-signature
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "cgr.dev/chainguard/*"
      attestations:
      - name: cosign-signature
        attestationProvider: cosign
        attestationType: cosign
```

**Policy: Prevent image pull from untrusted registries**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-registries
spec:
  validationFailureAction: enforce
  rules:
  - name: validate-registries
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Only images from approved registries are allowed"
      pattern:
        spec:
          containers:
          - image: "cgr.dev/* | gcr.io/* | quay.io/chainguard/*"
```

### Pillar 3: Configuration & Metadata → Security Context

**Policy: Non-root user required**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  rules:
  - name: check-runAsNonRoot
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Containers must run as non-root user (65532 for Chainguard)"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
```

**Policy: Resource limits required (prevent DoS)**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-limits
spec:
  validationFailureAction: enforce
  rules:
  - name: cpu-memory-limits
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "CPU and memory limits must be set"
      pattern:
        spec:
          containers:
          - resources:
              limits:
                cpu: "?*"
                memory: "?*"
```

### Pillar 4: Vulnerabilities → Image Scanning

**Policy: Scan images before deployment**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-image-scan
spec:
  validationFailureAction: enforce
  rules:
  - name: scan-with-trivy
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "*"
        scanImages: true
        scannerImages:
        - "aquasec/trivy:latest"
        expectedAttestations:
        - name: vulnerability-scan
          attestationType: sarif
          failIfNotFound: true
```

**Policy: Block images with critical/high CVEs**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-critical-cves
spec:
  validationFailureAction: enforce
  rules:
  - name: check-cves
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "*"
      attestations:
      - name: vuln-scan-results
        attestationProvider: grype
        expectedCVESeverity: "HIGH"  # Block high+ severity
```

## Implementation Strategy

### Phase 1: Audit & Educate (Week 1-2)

Deploy policies in **audit mode** (log violations, don't block):

```yaml
spec:
  validationFailureAction: audit  # Don't block, just log
```

**Outcome:** See what deployments violate policies, educate teams.

### Phase 2: Enforce Core Policies (Week 3-4)

Enforce the most critical policies:
- Non-root user
- Distroless/minimal images
- Drop capabilities
- Resource limits

```yaml
spec:
  validationFailureAction: enforce  # Now block violations
```

### Phase 3: Enforce Advanced Policies (Month 2)

Add more restrictive policies:
- Read-only root filesystem
- Image scanning
- Image signatures
- Seccomp profiles

### Rollout Example Timeline

```
Week 1-2: Audit mode
├─ require-non-root (audit)
├─ restrict-privileged (audit)
├─ drop-all-caps (audit)
└─ require-resource-limits (audit)

Week 3-4: Enforce basics
├─ require-non-root (enforce)
├─ restrict-privileged (enforce)
├─ drop-all-caps (audit → enforce)
└─ require-resource-limits (audit → enforce)

Month 2: Enforce advanced
├─ require-ro-root (audit → enforce)
├─ require-image-scan (enforce)
├─ allowed-registries (enforce)
└─ require-image-signature (enforce)

Month 3+: Continuous improvement
└─ Tighten policies, add org-specific rules
```

## Common Policy Patterns

### Pattern 1: Required vs. Optional

**For non-root (required):**
```yaml
validationFailureAction: enforce  # Block violations
```

**For read-only root (nice to have, being rolled out):**
```yaml
validationFailureAction: audit  # Log only
```

### Pattern 2: Exceptions (Override for specific workloads)

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  exceptions:
  - ruleNames:
    - check-runAsNonRoot
    resources:
      namespaces:
      - ci-cd  # CI/CD can use root for setup jobs
    operations:
    - create
```

### Pattern 3: Per-Namespace Policies

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: prod-strict
spec:
  validationFailureAction: enforce
  rules:
  - name: strict-prod
    match:
      resources:
        kinds:
        - Pod
        namespaceSelector:
          matchLabels:
            env: production  # Only prod
    validate:
      message: "Production workloads have strict requirements"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
            readOnlyRootFilesystem: true
          containers:
          - securityContext:
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL
```

## Monitoring & Reporting

### View policy violations

```bash
# See what was blocked
kubectl describe policyviolation -n default

# See audit logs
kubectl logs -n kyverno deployment/kyverno | grep "validation failed"
```

### Generate reports

```bash
# Violations by policy
kubectl get policyviolation --all-namespaces -o json | \
  jq '.items[] | {policy: .metadata.name, namespace: .metadata.namespace}' | \
  uniq -c | sort -rn
```

### Dashboard/metrics

Kyverno exports metrics:
- `kyverno_policy_violations_total` — Total violations
- `kyverno_policy_results_total` — Audit/enforce results
- `kyverno_policy_mutation_requests_total` — Policy mutations

Integrate into Prometheus/Grafana for visibility.

## Best Practices

### DO
- ✅ Start in audit mode, move to enforce gradually
- ✅ Include exemptions for known exceptions
- ✅ Document policy rationale
- ✅ Train teams on required configurations
- ✅ Monitor policy violations
- ✅ Review and update policies quarterly

### DON'T
- ❌ Enforce all policies at once (breaks everything)
- ❌ Create overly complex policies (use Kyverno not OPA initially)
- ❌ Ignore policy violations (investigate and fix)
- ❌ Add exceptions without tracking (creates loopholes)
- ❌ Never update policies (new threats emerge)

## Red Flags (Broken Policy Enforcement)

⚠️ **Problems you might have:**
- ❌ Policies in audit mode for months (never enforced)
- ❌ Wildcard exemptions (defeats the purpose)
- ❌ No one knows what policies exist
- ❌ Policies never updated for new CHPs
- ❌ No visibility into violations
- ❌ Teams working around policies instead of following them

## Implementation Checklist

### Planning
- [ ] Policies aligned with CHPs framework
- [ ] Pilot namespace identified (non-critical workloads)
- [ ] Teams notified of upcoming changes
- [ ] Audit period scheduled (1-2 weeks)

### Deployment
- [ ] Kyverno or OPA/Gatekeeper installed
- [ ] Policies deployed in audit mode
- [ ] Monitoring/alerting configured
- [ ] Team trained on required configurations

### Enforcement
- [ ] Violations reviewed and understood
- [ ] Teams update configurations to comply
- [ ] Policies switched from audit to enforce
- [ ] Exception handling documented

### Ongoing
- [ ] Violation metrics monitored
- [ ] Monthly policy review
- [ ] New policies added as threats emerge
- [ ] Team feedback collected

## References

- **Kyverno Documentation:** https://kyverno.io/docs/
- **Chainguard Security Policies:** https://www.chainguard.dev/supply-chain-security-101/
- **OPA/Gatekeeper:** https://open-policy-agent.org/
- **Kubernetes Security Context:** https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
- **Chainguard Academy:** https://edu.chainguard.dev/
- **NIST Container Security:** https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf
