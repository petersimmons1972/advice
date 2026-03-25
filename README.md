# 🛡️ Advice — The Open-Source Security Guidance Repository

> **A unified, vendor-agnostic repository of security best practices, threat intelligence, and enforceable policies.**
> Collecting wisdom from Chainguard, NIST, AWS, GCP, GitHub, and the broader security community.

**STATUS:** 🟢 **Active & Growing**
**License:** Vendor guidance (see each vendor directory for source licensing)
**Community:** Open to contributions

---

## Why This Project Exists

### The Problem

Security is **fragmented and contradictory.**

- Chainguard says: *Daily image rebuilds*
- AWS says: *Scan on each deployment*
- NIST says: *Establish baseline*
- Your team says: *We have a deadline*

Result: **Security guidance lives in scattered blog posts, PDFs, and Slack threads.** When a new threat emerges (LiteLLM malware, anyone?), you don't know:
- ❌ Do we have this dependency?
- ❌ How quickly can we patch?
- ❌ What's our SLO for critical CVEs?
- ❌ Who do we notify?

### The Solution

**One repository. Multiple vendors. Clear guidance.**

Advice collects security wisdom from authoritative sources, reconciles conflicts, and makes policies machine-readable and enforceable.

---

## What's Inside

### 🚨 Phase 1: Vendor Guidance (Active)

We collect security recommendations from trusted sources and organize them so you can:
- **Compare approaches** — See how different vendors solve the same problem
- **Make informed decisions** — Understand trade-offs and rationale
- **Build policies** — Turn guidance into enforceable rules

```
┌─────────────────────────────────────────────────────────────────┐
│                    VENDOR GUIDANCE (Phase 1)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Chainguard ──────┐                                             │
│  (Container       │                                             │
│   Hardening)      │                                             │
│                   ├──→ Consolidated Security Framework          │
│  NIST ────────────┤    (Reconciled, Authoritative, Traceable) │
│  (Framework)      │                                             │
│                   ├──→ Used by:                                 │
│  AWS ─────────────┤    • Clearwatch (EDR reports)              │
│  (Cloud)          │    • Infrastructure (K8s policies)         │
│                   │    • Homelab (image hardening)             │
│  GCP ─────────────┤                                             │
│  (Cloud)          │                                             │
│                   │                                             │
│  GitHub ──────────┘                                             │
│  (Supply Chain)                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 🎯 Phase 2: Enforceable Policies (TBD)

Synthesize vendor guidance into machine-readable policies:
- JSON/YAML security standards
- OPA/Kyverno enforcement rules
- Terraform compliance checks
- Audit trails linking policies to sources

```
Vendor Guidance (Multi-perspective)
            ↓
Conflict Resolution (Decision-making)
            ↓
Policy Definition (Machine-readable)
            ↓
Enforcement (Admission control, CI/CD)
            ↓
Audit & Monitoring (Compliance tracking)
```

---

## Current Content

### ✅ Chainguard Guidance (Complete)

5 comprehensive guides on container security:

| Guide | Focus | Key Topics |
|-------|-------|-----------|
| **Container Hardening Priorities** | Security framework | 4-pillar approach (Minimalism, Provenance, Config, Vulns) |
| **Distroless Patterns** | Image design | Multi-stage builds, debugging strategies, trade-offs |
| **Supply Chain Security** | Dependencies & patches | SBOM, cosign signing, automation, CVE response |
| **Vulnerability Management** | Detection to response | Scanning tools, SLOs, risk assessment, patching |
| **Policy Enforcement** | Admission control | Kyverno/OPA, CHP mapping, rollout strategy |

### 🚨 Security Alerts (Active)

- **LiteLLM Supply Chain Attack** — Malicious versions 1.82.7-1.82.8 steal all credentials. Pin to 1.82.6.

### 📋 Multi-Vendor Structure (Ready)

Directories prepared for:
- ✅ Chainguard (5 guides complete)
- 🔄 NIST (Framework)
- 🔄 AWS (Cloud security)
- 🔄 GCP (Cloud security)
- 🔄 GitHub (Supply chain)

---

## Real-World Example: Responding to LiteLLM

### Scenario: Your team learns about LiteLLM malware

**Without Advice:**
```
Team: "Do we have LiteLLM?"
Engineer 1: "Grep the Dockerfiles..."
Engineer 2: "Check requirements.txt manually..."
Nobody: "We don't know version numbers..."
CISO: "Are we compromised?"
Everyone: "Unknown. Start rotating credentials."
⏱️ Response time: 4-6 hours
```

**With Advice:**

1. **Check threat database** → Find `/security-advisories/litellm-supply-chain.md`
   ```
   Versions 1.82.7+ COMPROMISED
   Mitigation: Pin to 1.82.6 or earlier
   ```

2. **Query SBOM** (generated via Phase 2 policies)
   ```bash
   $ grep litellm *.sbom.json
   image-1: litellm==1.82.6 ✅ SAFE
   image-2: litellm==1.82.8 ❌ COMPROMISED
   ```

3. **Follow supply-chain-security guidance**
   - Identify affected deployments
   - Rebuild with safe version
   - Verify with scanning
   - Rotate credentials
   - Deploy patches

4. **Track in audit log**
   ```json
   {
     "threat": "litellm-supply-chain",
     "source": "advice/security-advisories/litellm-supply-chain.md",
     "decision": "pin to 1.82.6",
     "deployed": "2026-03-25T02:30:00Z"
   }
   ```

⏱️ Response time: **30 minutes**

---

## How to Use This Repository

### 👀 I'm Looking for Security Guidance

1. **Start with the threat you care about**
   - Container security? → `vendors/chainguard/`
   - Vulnerability management? → `vendors/chainguard/vulnerability-management.md`
   - Supply chain attacks? → `security-advisories/` + `vendors/chainguard/supply-chain-security.md`

2. **Browse by vendor for different perspectives**
   - Chainguard: Implementation-focused, distroless-first
   - NIST: Framework-focused, compliance-oriented
   - AWS: Cloud-centric, managed-service patterns
   - GitHub: Source control & CI/CD security

3. **Check cross-vendor topics** (coming in Phase 2)
   - `topics/container-security/` — How different vendors approach the same problem

### 🔧 I'm Building Security Policies (Phase 2)

1. Find the vendor guidance for your domain
2. Check `policies/` to see if it's been synthesized already
3. If not, follow the `policies/README.md` to understand the policy format
4. Map vendor recommendations to enforceable rules

### 📝 I Want to Contribute

**Add a new vendor:**
1. Create directory: `vendors/<vendor-name>/`
2. Add guides with source citations
3. Follow naming convention: `<topic>.md`
4. PR with title: `docs: add <vendor> guidance — <topics>`

**Found a threat?**
1. Create file: `security-advisories/<threat-name>.md`
2. Include impact, mitigation, sources
3. Cross-reference from relevant vendor docs
4. Mark severity with 🚨 **CRITICAL** if urgent

**Disagree with guidance?**
1. Check the rationale in the document
2. Review the source material
3. Open an issue explaining your perspective
4. We'll document the alternative approach

---

## The Vision: From Fragmented to Unified

### Today (Fragmented)

```
Your team:
├─ Reads Chainguard blog
├─ Watches AWS re:Invent talks
├─ Reads NIST PDF (skims it)
├─ Checks GitHub Dependabot email
├─ Googles when threat drops
└─ Makes decisions without context

Result: Inconsistent, reactive security
```

### Tomorrow (Unified)

```
Your team:
├─ Opens Advice repo
├─ Finds guidance for your use case
├─ Sees how multiple vendors approach it
├─ Understands reconciled policy
├─ Applies policy via code (Kyverno/OPA)
└─ Audit trail shows why each decision was made

Result: Consistent, proactive, auditable security
```

---

## Highlights: What Makes Advice Different

| Aspect | Traditional | Advice |
|--------|-----------|--------|
| **Organization** | Vendor blogs scattered | Single repo, vendor-organized |
| **Conflicts** | Vendors contradict each other | Documented & reconciled decisions |
| **Traceability** | Blog post link lost in Slack | Source always cited, audit trail |
| **Enforcement** | Manual checklists | Machine-readable policies |
| **Updates** | When you remember | Automated, tracked as commits |
| **Community** | Vendor silos | Multi-vendor perspective |
| **Trust** | Hope vendors know best | Compare, understand, decide |

---

## Quick Navigation

### 🆘 I Have a Problem

| Problem | Solution |
|---------|----------|
| CVE dropped, need to patch | `vendors/chainguard/vulnerability-management.md` |
| What's exploitable in my image? | `vendors/chainguard/vulnerability-management.md#risk-assessment` |
| How do I enforce security policies? | `vendors/chainguard/policy-enforcement.md` |
| Dealing with supply chain attack | `vendors/chainguard/supply-chain-security.md` |
| Building distroless images | `vendors/chainguard/distroless-patterns.md` |
| Suspicious package, is it malware? | `security-advisories/` |

### 📚 I Want to Learn

| Topic | Where to Start |
|-------|-------|
| Container security | `vendors/chainguard/container-hardening-priorities.md` |
| Supply chain safety | `vendors/chainguard/supply-chain-security.md` |
| Vulnerability management | `vendors/chainguard/vulnerability-management.md` |
| Kubernetes policies | `vendors/chainguard/policy-enforcement.md` |
| Best practices from multiple vendors | (Phase 2: `topics/` — coming soon) |

---

## Repository Stats

```
Project Status:          🟢 Active
Vendors Covered:         🟢 Chainguard (complete)
                         🔄 NIST, AWS, GCP, GitHub (queued)
Security Advisories:     🚨 1 active (LiteLLM)
Phase 1 Progress:        ████████░░ 80%
Phase 2 Progress:        ██░░░░░░░░ 20%
Last Updated:            2026-03-25
Contributors:            Open to community
```

---

## Key Principles

### 🎯 **Vendor-Agnostic**
We don't favor one vendor. We collect wisdom from all, document conflicts, make conscious decisions.

### 🔍 **Transparent**
Every recommendation is sourced. You can trace back to the original vendor guidance and understand *why* we made each decision.

### 🚀 **Actionable**
Guidance isn't abstract principles. It includes concrete SLOs, tools, example configurations, and rollout strategies.

### 📊 **Machine-Readable**
Phase 1 is human-readable (Markdown). Phase 2 will be machine-readable (JSON/YAML/Rego) so policies can be automated.

### 🛡️ **Threat-Focused**
When security threats emerge (LiteLLM, Log4Shell, etc.), we document them with detection and response guidance.

---

## Contributing

### Types of Contributions

✅ **Add vendor guidance**
- Research and document best practices from Chainguard, NIST, AWS, GCP, GitHub, etc.
- Include examples, trade-offs, and source citations

✅ **Report security threats**
- Find zero-days or supply chain attacks?
- Document detection and mitigation
- Include severity, affected versions, and references

✅ **Improve documentation**
- Clarify confusing sections
- Add examples or diagrams
- Fix links or outdated information

✅ **Reconcile vendor conflicts**
- When vendors disagree, document both approaches
- Propose reconciled decision with rationale

❌ **No vendor marketing**
- We're not promoting Chainguard (or any vendor)
- If guidance is good, we include it
- If guidance is incomplete, we note it

### How to Contribute

1. **Fork and clone**
   ```bash
   git clone https://github.com/yourusername/advice.git
   cd advice
   ```

2. **Create a branch**
   ```bash
   git checkout -b add-nist-framework
   ```

3. **Add your content**
   - Follow existing format and structure
   - Include source citations
   - Add to appropriate vendor directory

4. **Commit and push**
   ```bash
   git commit -m "docs: add NIST cybersecurity framework guidance"
   git push origin add-nist-framework
   ```

5. **Open a pull request**
   - Explain what vendor guidance you're adding
   - Link to source materials
   - Mention if this fills a gap or reconciles vendor conflict

---

## Similar Projects & Inspiration

- **SLSA Framework** (Supply chain security) → https://slsa.dev/
- **NIST Cybersecurity Framework** → https://www.nist.gov/cyberframework
- **Cloud Security Alliance** → https://cloudsecurityalliance.org/
- **OWASP** (Web security focus) → https://owasp.org/
- **CIS Benchmarks** (Standards) → https://www.cisecurity.org/

**Advice differs:** We focus on *actionable, multi-vendor guidance* rather than abstract standards or vendor marketing.

---

## License & Attribution

Each vendor's guidance retains its original license. When citing guidance:

> "This recommendation comes from [Vendor] and is documented in [guide].
> Original source: [URL]"

---

## Contact & Questions

**Questions about the project?**
- Open an issue on GitHub
- Include your question and context

**Want to contribute?**
- Check existing guidance first
- Start small (one guide per PR)
- We'll review and provide feedback

**Found an error or outdated info?**
- Open an issue with sources
- We'll prioritize security-related fixes

---

## Acknowledgments

This project draws wisdom from:
- **Chainguard** — Container security expertise and daily image rebuilds
- **NIST** — Cybersecurity framework and standards
- **AWS** — Cloud security best practices
- **GCP** — Distroless foundation and vulnerability research
- **GitHub** — Supply chain security and advisories
- **Security research community** — Threat intelligence and incident response

---

## The Ask

We're building a resource that saves teams time and improves security outcomes. If Advice helps you:

⭐ **Star the repository** — Helps others discover it
📢 **Share with your team** — Let's make security guidance accessible
🤝 **Contribute guidance** — Help us expand to more vendors
🐛 **Report errors** — Help us stay accurate and current

**Together, we can make security less fragmented and more unified.**

---

**Last updated:** 2026-03-25
**Repository:** https://github.com/petersimmons1972/advice
**Status:** 🟢 Active & accepting contributions
