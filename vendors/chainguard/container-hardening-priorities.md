# Chainguard: Container Hardening Priorities (CHPs) Framework

**Source:** Chainguard Academy
**Last updated:** 2026-03-25
**Scope:** Container security best practices across all workloads

## Overview

Container Hardening Priorities (CHPs) provide a structured, auditable framework for evaluating and implementing container security. Rather than treating security as a checkbox, CHPs organize practices into four pillars that work together.

**Philosophy:** Minimize surface area → Maximize observability → Enforce controls → Respond to threats

## The Four Pillars

### 1. Minimalism — Reduce Attack Surface

**Principle:** Every unnecessary file, package, or capability is a potential vulnerability.

**Practices:**
- **Distroless base images** — Remove package managers, shells, debuggers
  - Eliminates entire classes of vulnerabilities (no /bin/sh to escape from)
  - Reduces CVE count by ~90% compared to full-OS images
  - Example: Chainguard distroless has 0-2 CVEs vs. Ubuntu base with 50+
- **Single-purpose containers** — One application per container
  - No sidecar admin tools or "just in case" utilities
  - Reduces blast radius if one tool is compromised
- **Multi-stage builds** — Separate build environment from runtime
  - Build stage: full dev tools (gcc, git, etc.)
  - Runtime stage: only application + minimal runtime
  - Example: Go binary doesn't need the Go compiler in production

**Trade-offs:**
- ❌ Harder to debug in production (no shell access)
- ❌ Requires more discipline in image design
- ✅ Dramatically lower vulnerability count
- ✅ Faster image downloads and startup
- ✅ Smaller attack surface for exploits

### 2. Provenance — Know Your Supply Chain

**Principle:** Every artifact must be traceable to a known source.

**Practices:**
- **Signed images** — Cryptographic verification of image authenticity
  - Prevents man-in-the-middle injection during pull
  - Verifiable through cosign or similar tools
- **Bill of Materials (SBOM)** — Complete manifest of what's in the image
  - Know every package, version, and license
  - Enables rapid vulnerability assessment when CVE drops
  - Tools: syft, grype
- **Source tracking** — Know which repo commit, branch, and tag produced this image
  - Enables reproducibility
  - Enables rollback to known-good versions
- **Build attestation** — Verify the image was built by expected CI/CD pipeline
  - Prevents supply chain attacks (rogue builds)
  - SLSA framework alignment

**Trade-offs:**
- ❌ Requires CI/CD integration and key management
- ❌ Slows initial setup
- ✅ Enables rapid incident response
- ✅ Catches supply chain compromises early

### 3. Configuration & Metadata — Control Behavior

**Principle:** Even minimal images need strict runtime configuration.

**Practices:**
- **Non-root user** — Application runs as unprivileged UID
  - Limits damage if process is compromised
  - Prevents many privilege escalation attacks
  - Common UID: 65532 (nonroot user in Chainguard)
- **Read-only root filesystem** — `/` is mounted read-only
  - Prevents persistence attacks
  - Forces attacker to use memory (volatile)
  - Requires explicit tmpfs/emptyDir mounts for writable paths
- **No privileged capabilities** — Drop all unnecessary Linux capabilities
  - Default: drop CAP_ALL, add back only what's needed
  - Example: web server doesn't need CAP_SYS_ADMIN
- **Security policies** — Enforce via OPA, Kyverno, SELinux
  - Prevent privileged containers
  - Enforce read-only root
  - Restrict syscalls (seccomp)

**Trade-offs:**
- ❌ Requires testing to identify needed capabilities
- ❌ More complex deployment configuration
- ✅ Dramatically limits post-compromise damage
- ✅ Detects misconfiguration before production

### 4. Vulnerabilities — Minimize & Respond Fast

**Principle:** Even hardened images need fast patching.

**Practices:**
- **Daily rebuilds** — Images rebuilt every 24h with latest patches
  - Zero-day window is minimized
  - Security patches deployed immediately on release
  - Chainguard rebuilds all images daily automatically
- **Vulnerability scanning** — Scan before push, scan before deploy
  - Tools: Grype, Trivy, Snyk
  - Catch regressions in CI/CD
  - Catch known-bad images before production use
- **Rapid update cadence** — Consumers update digests weekly or more
  - Renovate bot: automated weekly updates
  - Digestabot: Chainguard-specific bot
  - Manual: review and update on security events
- **CVE response SLO** — Patch critical/high within 24 hours
  - Chainguard images: typically patched within hours
  - Compare to distros: Ubuntu LTS may take weeks

**Trade-offs:**
- ❌ Requires robust update infrastructure
- ❌ Risk of regressions from updates
- ✅ Dramatically shorter time-to-patch
- ✅ Fewer long-lived vulnerabilities in production

## How CHPs Work Together

```
Minimalism (fewer packages)
    ↓
    → Fewer CVEs to track (Pillar 4 is easier)
    → Smaller build time, faster pulls (Pillar 1 payoff)

Provenance (track sources)
    ↓
    → Know if image contains CVE (Pillar 4 input)
    → Can reproduce/rollback to safe version (incident response)

Configuration (run safely)
    ↓
    → Even if compromised, damage is limited (containment)
    → Read-only root prevents persistence (incident response)

Vulnerabilities (patch fast)
    ↓
    → Minimalism means fewer to patch (Pillar 1 payoff)
    → Provenance lets us track which images need patching
    → Configuration limits exposure window (Pillar 3 payoff)
```

## Example: Chainguard Images vs. Ubuntu

| Metric | Chainguard (distroless) | Ubuntu 22.04 |
|--------|------------------------|------------|
| CVEs at pull | 0-2 | 40+ |
| Image size | ~20MB | 77MB |
| Non-root user | ✅ Yes (65532) | ❌ No |
| Read-only root | ✅ Recommended | ❌ No |
| Daily rebuild | ✅ Yes | ❌ No |
| Time to patch critical CVE | Hours | Days to weeks |
| Package manager in runtime | ❌ No | ✅ Yes (attack surface) |

## Implementation Checklist

### Minimalism
- [ ] Use distroless or minimal base image (Chainguard, Google Distroless, Alpine)
- [ ] Multi-stage build: dev stage with tools, runtime stage minimal
- [ ] Verify final image has only application + runtime dependencies
- [ ] No shells, package managers, or debug tools in production image
- [ ] Test with `docker run --rm <image> /bin/sh` → fails (correct)

### Provenance
- [ ] Images signed with cosign or similar
- [ ] SBOM generated and attached to image
- [ ] Image tagged with source commit/tag
- [ ] Build attestation configured in CI/CD
- [ ] Verify chain: commit → build → image → registry

### Configuration
- [ ] Non-root user (typically 65532 for Chainguard)
- [ ] Read-only root filesystem (/ mounted read-only)
- [ ] Drop all capabilities, add back only what's needed
- [ ] Security policy deployed (OPA/Kyverno/SELinux)
- [ ] Seccomp profile configured (if using Seccomp)

### Vulnerabilities
- [ ] Grype/Trivy scanning in CI/CD (before push)
- [ ] Vulnerability scanning at deployment (before run)
- [ ] Update bot configured (Renovate/Digestabot)
- [ ] SLO for critical/high patch defined
- [ ] Image rebuild frequency defined (recommend daily)

## References

- **Chainguard Academy:** https://edu.chainguard.dev/chainguard/chainguard-images/about/zerocve/
- **Container Security Best Practices:** https://www.chainguard.dev/supply-chain-security-101/container-security-best-practices-without-the-toil
- **Chainguard Images Overview:** https://edu.chainguard.dev/chainguard/chainguard-images/overview/
- **SLSA Framework:** https://slsa.dev/
- **cosign (image signing):** https://github.com/sigstore/cosign
- **Grype (vulnerability scanning):** https://github.com/anchore/grype
- **Kyverno (policy engine):** https://kyverno.io/
