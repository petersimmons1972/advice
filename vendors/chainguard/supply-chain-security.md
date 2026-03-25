# Chainguard: Supply Chain Security & Patch Management

**Source:** Chainguard Academy, Container Security Best Practices
**Last updated:** 2026-03-25
**Scope:** Managing dependencies, vetting sources, and responding to patches safely

## Overview

Supply chain security in containers means:
1. **Know what you have** — Complete inventory of dependencies
2. **Know where it came from** — Verify source authenticity
3. **Know when it's broken** — Track CVEs and vulnerabilities
4. **Know how to fix it** — Safe, auditable patch process

**Critical:** Supply chain attacks (like LiteLLM) happen when you lose track of where artifacts come from or can't quickly identify what needs patching.

## Pillar 1: Know What You Have (SBOM)

### What Is an SBOM?

A Software Bill of Materials (SBOM) is a complete, machine-readable manifest of every dependency in your image:
- Every package (openssl, python, libc, etc.)
- Every version number
- Every CVE that affects it
- Transitive dependencies (dependencies of dependencies)

### Why SBOM Matters

**Scenario:** A critical CVE drops for OpenSSL
- ❌ **Without SBOM:** "Do we have OpenSSL? Which version? Is it vulnerable?" (hours to answer)
- ✅ **With SBOM:** Query the manifest, get instant answer in seconds

**Scenario:** Supply chain attack (LiteLLM-style)
- ❌ **Without SBOM:** Attacker inserts malware, you don't know what changed
- ✅ **With SBOM:** Diff SBOMs before/after, see exactly what was added

### Tools

**syft** — Generate SBOMs from container images
```bash
syft cgr.dev/chainguard/python:latest -o json > python.sbom.json
```

**Attach to image** (with cosign):
```bash
syft cgr.dev/chainguard/python:latest -o spdx > python.sbom.spdx
cosign attach sbom --sbom python.sbom.spdx cgr.dev/chainguard/python:latest
```

**Query for dependencies:**
```bash
# List all packages
syft cgr.dev/chainguard/python:latest -o table

# Export as JSON for programmatic use
syft cgr.dev/chainguard/python:latest -o json | jq '.artifacts[] | select(.name=="openssl")'
```

### SBOM Formats

- **SPDX** — Machine-readable, most complete, standardized
- **CycloneDX** — Similar to SPDX, also standardized
- **JSON** — Easy to parse programmatically

**Recommendation:** Store as SPDX attached to image with cosign.

## Pillar 2: Know Where It Came From (Provenance)

### Verify Image Authenticity

Every image pull should verify:
1. **Signed by expected builder** — Cryptographic proof it came from your CI/CD
2. **Built from known commit** — Reproducible from source code
3. **No tampering in transit** — Hash verification

### Tools & Patterns

**cosign** — Sign and verify container images
```bash
# Sign an image
cosign sign --key cosign.key cgr.dev/mycompany/myapp:latest

# Verify signature
cosign verify --key cosign.pub cgr.dev/mycompany/myapp:latest

# Policy: require signature in Kubernetes
# (via Kyverno or admission controller)
```

**SLSA Framework** — Supply Level Security Assurance
- Level 1: Provenance available (what was built)
- Level 2: Version control + CI/CD logging (audit trail)
- Level 3: Hermetic builds (no external inputs during build)
- Level 4: Reproducible builds (exact same output every time)

**Build attestation** — Record what went into the build
```bash
# Example: record commit, builder, timestamp
cosign attest --attestation ./provenance.json cgr.dev/mycompany/myapp:latest
```

### Chainguard's Approach

Chainguard images come with:
- ✅ Signed images (cosign)
- ✅ SBOM attached (syft)
- ✅ Build attestation available
- ✅ Reproducible builds (SLSA Level 3)
- ✅ Source traceability (linked to GitHub commits)

When you pull `cgr.dev/chainguard/python:latest`, you can verify it was built by Chainguard's CI/CD from known source.

## Pillar 3: Know When It's Broken (CVE Tracking)

### Vulnerability Scanning

**grype** — Scan images for known CVEs
```bash
# Scan image
grype cgr.dev/chainguard/python:latest

# Output: list of CVEs, severity, CVSS score
# Example:
# python          3.11.5  High   CVE-2023-12345   CVSS 7.8
# openssl         3.0.8   Critical CVE-2023-54321  CVSS 9.1
```

**trivy** — Similar tool, often integrated in Kubernetes
```bash
# Scan image
trivy image cgr.dev/chainguard/python:latest

# Scan with policy (fail on high severity)
trivy image --severity HIGH,CRITICAL cgr.dev/chainguard/python:latest
```

### Scanning at Multiple Points

1. **Build time** (in CI/CD pipeline)
   ```bash
   # Dockerfile
   RUN apk add --no-cache curl
   # CI/CD scans before push
   grype . --fail-on high
   ```

2. **Registry time** (before pushing to registry)
   ```bash
   # Post-build hook
   grype myimage:latest || { echo "CVEs found"; exit 1; }
   ```

3. **Deploy time** (before running in production)
   ```bash
   # Kubernetes admission controller (Kyverno)
   # Blocks deployment if image has critical CVEs
   ```

4. **Runtime** (continuous monitoring)
   ```bash
   # Falco, Sysdig, or similar
   # Monitor for exploitation attempts
   ```

### CVE Response SLO

**Chainguard's SLO:**
- Critical CVEs: patched within 24 hours (often same-day)
- High CVEs: patched within 72 hours
- Medium/Low: patched in regular update cycle

**Your SLO should be:**
- Critical: deploy within 24 hours of patch available
- High: deploy within 1 week
- Medium: deploy within 1 month
- Low: deploy in next regular update

## Pillar 4: Know How to Fix It (Patch Management)

### Safe Patching Process

**Step 1: Alert on new CVE**
- Subscribe to CVE feeds (NVD, GitHub Security, vendor advisories)
- Automated alerts when your dependencies are affected
- Tools: Dependabot, Snyk, Grype with alert integration

**Step 2: Assess impact**
```bash
# Which of our images are affected?
grype --input sbom python.sbom.json | grep CVE-2023-12345

# What's the severity?
curl https://nvd.nist.gov/vuln/detail/CVE-2023-12345
```

**Step 3: Get patch**
- Chainguard rebuilds daily → patches available within hours
- Alternative: wait for distro patch (days to weeks)
- Fallback: apply patch manually if upstream slow

**Step 4: Update image digest**
```bash
# Old (vulnerable)
FROM cgr.dev/chainguard/python@sha256:abc123...

# New (patched) - update digest
FROM cgr.dev/chainguard/python@sha256:def456...
```

**Step 5: Test**
- Run image through scanning again (verify CVE fixed)
- Run tests on updated image
- Deploy to staging
- Verify functionality

**Step 6: Deploy**
- Update production digest
- Monitor for any regressions
- Document patch in changelog

### Automating Patches

**Renovate Bot** (for any images)
```json
{
  "extends": ["config:base"],
  "docker": {
    "enabled": true,
    "automerge": false,
    "minimumReleaseAge": "7 days"
  },
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"]
  }
}
```

Creates pull requests when:
- New image digest available
- CVE found in current digest
- Updates security advisories

**Digestabot** (Chainguard-specific)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: digestabot-config
data:
  config.yaml: |
    chainguard-images:
      python:
        update-frequency: daily
        alert-on-cve: true
        auto-update-critical: true
```

Automatically updates digests in your configs when Chainguard rebuilds.

## Case Study: Responding to LiteLLM Attack

**Scenario:** LiteLLM malware published (versions 1.82.7-1.82.8)

### With Good Supply Chain Practices

1. **SBOM exists** → Immediately identify which images include litellm
2. **Scanning enabled** → CVE database updated, scan flags the malware
3. **Signed images** → Verify whether malicious version was pulled
4. **Patch SLO** → Remove 1.82.7+, rebuild with 1.82.6 within hours
5. **Audit trail** → Track which deployments received malware version
6. **Response** → Rotate credentials, update images, investigate

**Time to detect & respond:** Hours

### Without Supply Chain Practices

1. "Do we have litellm?" — Manual grepping through Dockerfiles
2. "Which version?" — Unknown, unclear
3. "Which deployments?" — Check each manually
4. "Did we get the malicious version?" — Unknown
5. "Are credentials compromised?" — Have to assume yes, rotate everything

**Time to detect & respond:** Days

## Red Flags (Supply Chain Attack Indicators)

Watch for:
- ❌ Unexpected packages in SBOM (new dependency appeared)
- ❌ Version jump (1.82.6 → 1.82.9 skipping release notes)
- ❌ Signature mismatch (image signed by different key)
- ❌ Source divergence (image doesn't match source commit)
- ❌ New CVEs suddenly appearing (malware detected by scanner)
- ❌ Unusual network traffic (exfiltration)
- ❌ Unauthorized contributors to image repo

## Implementation Checklist

### Know What You Have
- [ ] SBOM generated for all images
- [ ] SBOM attached to images (cosign)
- [ ] SBOM queryable (can identify dependencies quickly)
- [ ] Dependency inventory maintained

### Know Where It Came From
- [ ] Images signed with known key
- [ ] Signature verification enforced (admission controller)
- [ ] Build attestation available
- [ ] Source commit linked to image tag

### Know When It's Broken
- [ ] Scanning enabled at build time
- [ ] Scanning enabled at deploy time
- [ ] CVE alert subscriptions active
- [ ] Grype/Trivy integrated in CI/CD
- [ ] Policy: fail on critical CVEs

### Know How to Fix It
- [ ] SLO defined for critical/high/medium patches
- [ ] Patch testing process documented
- [ ] Automation (Renovate/Digestabot) configured
- [ ] Deployment process for patches defined
- [ ] Incident response runbook for supply chain attacks

## Tools Reference

| Tool | Purpose | Integration |
|------|---------|-----------|
| syft | Generate SBOM | CI/CD, post-build |
| grype | Scan for CVEs | CI/CD, registry, Kubernetes |
| trivy | Scan for CVEs | CI/CD, registry, Kubernetes |
| cosign | Sign/verify images | Registry, admission controller |
| Renovate | Auto-update digests | GitHub, GitLab, CI/CD |
| Digestabot | Auto-update Chainguard | Kubernetes, Helm |
| Kyverno | Policy enforcement | Kubernetes admission |
| SLSA | Supply chain assurance | CI/CD assessment |

## References

- **Chainguard Supply Chain Security:** https://www.chainguard.dev/supply-chain-security-101/
- **SBOM with syft:** https://github.com/anchore/syft
- **Vulnerability scanning with grype:** https://github.com/anchore/grype
- **Image signing with cosign:** https://github.com/sigstore/cosign
- **SLSA Framework:** https://slsa.dev/
- **Renovate bot:** https://www.whitesourcesoftware.com/free-developer-tools/renovate/
- **NVD CVE Database:** https://nvd.nist.gov/
- **GitHub Security Advisories:** https://github.com/advisories
