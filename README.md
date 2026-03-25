# Advice — Security & Best Practices Repository

**STATUS: active**

## Overview

Central repository for security guidance, best practices, and threat intelligence from authoritative sources. Curated advice on infrastructure, supply chain security, container hardening, and vulnerability management.

## Purpose

**Phase 1 — Collect:** Single source of truth for security recommendations from multiple vendors
- **Cross-project reference** — Clearwatch, infrastructure, homelab, and other projects pull guidance from here
- **Threat tracking** — Critical security notices (LiteLLM, supply chain attacks, CVEs)
- **Multi-vendor perspectives** — Container security, secrets management, image hardening from Chainguard, NIST, AWS, GCP, GitHub, etc.

**Phase 2 — Consolidate:** Synthesize vendor guidance into enforceable security policies
- **JSON/YAML policies** — Machine-readable security standards
- **Vendor reconciliation** — Document conflicts and record decisions
- **Implementation** — OPA/Kyverno/Terraform enforcement rules
- **Audit trail** — Track source recommendations for each policy decision

## Directory Structure

```
advice/
├── README.md                    # This file
├── CLAUDE.md                    # Project instructions
├── security-advisories/         # Critical security notices and alerts
│   ├── litellm-supply-chain.md  # LiteLLM malware (versions 1.82.7-1.82.8)
│   └── ...
├── vendors/                     # Guidance organized by source vendor/author
│   ├── chainguard/              # Chainguard resources and guidance
│   │   ├── container-hardening.md   # CHPs framework and best practices
│   │   ├── distroless-patterns.md   # Distroless image design patterns
│   │   └── vulnerability-management.md
│   ├── nist/                    # NIST Cybersecurity Framework
│   ├── aws/                     # AWS security best practices
│   ├── gcp/                     # Google Cloud Platform security
│   ├── github/                  # GitHub security advisories
│   └── [other-vendors]/
├── topics/                      # Cross-vendor guidance by topic
│   ├── container-security/      # General container security practices
│   │   ├── image-scanning.md        # Vulnerability scanning (Grype, Trivy)
│   │   ├── update-strategies.md     # Keeping images current (Renovate, Digestabot)
│   │   └── policy-enforcement.md    # OPA Gatekeeper, Kyverno patterns
│   └── supply-chain/            # Supply chain security
│       └── dependency-vetting.md
└── cve-tracking/                # Known vulnerabilities and tracking
    └── active-threats.md        # Currently active exploited CVEs
```

## Quick Reference

- **LiteLLM threat:** Versions 1.82.7+ are compromised; pin to 1.82.6 or earlier
- **Chainguard advisories:** https://images.chainguard.dev/security
- **Container hardening:** Use distroless base images + daily updates + vulnerability scanning

## Contributing

When you discover security guidance, threats, or best practices:
1. Add the material to the appropriate directory
2. Cross-reference from the project that needs it
3. Commit with `docs: add [topic]` message

## References

- [Chainguard Academy](https://edu.chainguard.dev/)
- [Chainguard Images Security](https://images.chainguard.dev/security)
- [Container Security Best Practices](https://www.chainguard.dev/supply-chain-security-101/container-security-best-practices-without-the-toil)
