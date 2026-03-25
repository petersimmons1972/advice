# Security Policies — Consolidated & Enforceable

**STATUS: placeholder (Phase 2)**

## Overview

Machine-readable security policies synthesized from vendor guidance across `../vendors/`. These policies are:
- **Enforceable** — Can be implemented via OPA/Kyverno, policy-as-code tools
- **Consolidated** — Reconcile vendor differences into unified standards
- **Auditable** — Track source vendor recommendations for each policy decision
- **Versioned** — Changes documented with rationale

## Philosophy

When vendors conflict (e.g., Chainguard vs. AWS on image scanning frequency):
1. Document both positions with sources
2. Make a **conscious decision** on the combined policy
3. Record the rationale and trade-offs
4. Reference back to vendor docs for implementers

**Example conflict resolution:**
```json
{
  "policy": "vulnerability-scan-frequency",
  "vendors": {
    "chainguard": "daily (security advisory)",
    "aws": "on each deployment (best practices)"
  },
  "decision": "daily",
  "rationale": "Chainguard's daily rebuild + scanning catches zero-days faster. AWS guidance is baseline minimum.",
  "references": [
    "vendors/chainguard/vulnerability-management.md",
    "vendors/aws/image-scanning.md"
  ]
}
```

## Directory Structure

```
policies/
├── README.md                              # This file
├── container-security/                    # Container & image policies
│   ├── distroless-requirements.json       # Base image standards
│   ├── vulnerability-scanning.json        # Scanning and remediation SLOs
│   ├── image-update-frequency.json        # How often images rebuild/update
│   └── runtime-security.json              # Runtime detection and enforcement
├── supply-chain/                          # Dependency and artifact policies
│   ├── dependency-vetting.json            # Package validation requirements
│   ├── artifact-signing.json              # Signed artifacts enforcement
│   └── license-compliance.json            # License verification requirements
├── iam/                                   # Identity and access management
│   ├── least-privilege.json               # Min permission standards
│   ├── credential-rotation.json           # Secret rotation SLOs
│   └── audit-logging.json                 # What must be logged
├── secrets-management/                    # Secrets handling
│   ├── secret-storage.json                # Where/how secrets are stored
│   ├── secret-rotation.json               # Rotation frequency and process
│   └── secret-access-audit.json           # Access logging requirements
└── enforcement/                           # How policies are enforced
    ├── opa-rego/                          # OPA/Gatekeeper rules
    ├── kyverno/                           # Kyverno policies
    └── terraform/                         # Terraform policy-as-code
```

## Policy JSON Schema

(To be defined as we build Phase 2)

Each policy document will include:
- **Policy ID** — Unique identifier
- **Category** — container-security, supply-chain, iam, secrets-management
- **Description** — Human-readable policy statement
- **Vendors** — Which vendors recommend this, with sources
- **Requirements** — Specific, testable requirements
- **Enforcement** — How it's implemented (OPA, Kyverno, manual audit, etc.)
- **SLO** — Service level objective (e.g., "scan within 24h of image build")
- **Exceptions** — When/how policy can be waived
- **Last Updated** — Timestamp
- **Rationale** — Why this policy, trade-offs considered

Example (placeholder):
```json
{
  "policy_id": "CS-001",
  "category": "container-security",
  "title": "Distroless Base Images Required",
  "description": "All production container images must use distroless or minimal base images",
  "vendors": {
    "chainguard": "security advisory — distroless reduces CVE surface",
    "aws": "best practices — minimal attack surface",
    "nist": "supply chain security — minimize dependencies"
  },
  "requirements": [
    "No package manager in runtime image",
    "No shell interpreter",
    "Base image from curated list (Chainguard, Google Distroless, etc.)"
  ],
  "enforcement": {
    "tool": "kyverno",
    "file": "enforcement/kyverno/distroless-required.yaml"
  },
  "exceptions": [
    "Development/debug images (tagged with -debug)",
    "One-off admin tooling (documented in JIRA)"
  ],
  "rationale": "Balances security (99% of use cases) with operational reality (debug access when needed)"
}
```

## Workflow (Future)

1. **Research** — Vendor guidance in `../vendors/`
2. **Synthesis** — Team reviews conflicting recommendations
3. **Decision** — Record policy + rationale in JSON
4. **Implementation** — Wire into OPA/Kyverno/Terraform
5. **Audit** — Track which projects conform to policies
6. **Update** — As vendor guidance changes, update policies

## Expected Timeline

- **Near term:** Skeleton JSON schema + Chainguard-derived policies
- **Mid term:** Multi-vendor consolidation (Chainguard + NIST + AWS)
- **Long term:** Full policy library with enforcement across all projects (Clearwatch, infrastructure, homelab)

---

**Note:** This directory is currently a placeholder. Phase 2 begins when we're ready to synthesize vendor advice into enforceable policies.
