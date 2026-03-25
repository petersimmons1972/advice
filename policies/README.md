# Security Policies — Phase 2 Exploration

**STATUS: experimental (form TBD)**

## Overview

Consolidation of vendor guidance from `../vendors/` into a unified security policy framework. The **final form is unknown** — this directory is for exploration and experimentation.

Possible forms:
- **JSON/YAML policies** — Machine-readable standards
- **Markdown checklists** — Human-readable compliance checklists
- **OPA/Rego rules** — Policy-as-code enforcement
- **Terraform modules** — Infrastructure enforcement
- **Decision trees** — Flowcharts for policy decisions
- **A combination** — Different formats for different contexts
- **Something else entirely** — To be discovered

Key requirements (TBD which format best serves these):
- **Traceable** — Every policy decision links back to vendor sources
- **Reconciled** — Documents when vendors disagree and why we chose one approach
- **Auditable** — Can be reviewed and challenged
- **Actionable** — Implementers know what to do

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

## Potential Topics (Not Prescriptive)

Topics that will likely need policy consolidation:
- **Container security** — Base images, scanning, updates, runtime behavior
- **Supply chain** — Dependency vetting, artifact signing, license compliance
- **Identity & access** — Least privilege, credential rotation, audit logging
- **Secrets management** — Storage, rotation, access auditing
- **Threat response** — Incident classification, notification, containment
- **Compliance** — Regulatory requirements that override vendor preferences

## What We DON'T Know Yet

- **Format** — JSON? YAML? Markdown? Rego? All of the above?
- **Granularity** — One policy per rule? Per domain? Per vendor-conflict?
- **Enforcement** — Automated checks? Manual audits? Both?
- **Lifecycle** — How often do policies change? Who updates them?
- **Audience** — Developers? DevOps? Security teams? Auditors?
- **Integration** — How do Clearwatch/infrastructure/homelab consume policies?

## How Phase 2 Will Work

1. **Observe patterns** — As `../vendors/` fills up, patterns will emerge
   - Which recommendations appear across multiple vendors?
   - Where do vendors conflict?
   - Which topics are most critical?

2. **Experiment with form** — Try different formats to capture policies
   - A checklist format?
   - A decision table?
   - Executable code (OPA/Kyverno)?
   - Something hybrid?

3. **Iterate** — The format will become clear through use
   - What's easy to read?
   - What's easy to enforce?
   - What's easy to audit?
   - What survives first contact with Clearwatch/infrastructure/homelab?

## Key Constraint

**Vendor traceability** — Whatever form policies take, they must trace back to vendor sources:
- Which vendor recommended this?
- Did multiple vendors agree or disagree?
- Why did we choose this approach?

This is the non-negotiable requirement. The format is negotiable.

## Next Steps

1. **Fill `../vendors/`** with multi-vendor guidance (Chainguard, NIST, AWS, GCP, GitHub)
2. **Identify conflicts** — Where do vendors disagree? Why?
3. **Experiment** — Try different policy formats and see what works
4. **Discover form** — The final shape will emerge from actual use

## Notes

- This is **not** a placeholder awaiting a master plan
- This **is** a research area where the final form will be discovered through experimentation
- The one fixed constraint is **vendor traceability** — everything traces to sources
- Format decisions will be driven by "what works for Clearwatch, infrastructure, and homelab"
- We may end up with multiple formats for different contexts (e.g., checklist for humans, JSON for automation)
