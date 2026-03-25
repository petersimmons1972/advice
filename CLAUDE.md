# Advice Project Instructions

## Overview

This is a reference project for security guidance and best practices. It serves as a central knowledge base that other projects (Clearwatch, infrastructure, homelab) reference for authoritative security advice.

## What Goes Here

✅ **Curated security guidance** from authoritative sources (Chainguard, NIST, AWS, GCP, GitHub, security researchers)
✅ **Multi-vendor perspectives** — collect advice from all relevant security vendors
✅ **Critical security notices** that require immediate action (supply chain attacks, CVE alerts)
✅ **Best practices frameworks** (Container Hardening Priorities, distroless patterns, etc.)
✅ **Tool recommendations** with usage patterns (Grype, Trivy, Renovate, Kyverno)
✅ **Cross-project references** — when Clearwatch or infrastructure needs guidance, link here
✅ **Comparative analysis** — when vendors disagree, document different perspectives and rationales

❌ **Unverified claims** — everything must cite authoritative sources
❌ **Implementation code** — this is guidance documentation, not implementation
❌ **Project-specific details** — those stay in their respective project directories
❌ **Opinions without citation** — back recommendations with evidence
❌ **Single-vendor gospel** — encourage cross-vendor comparison and evaluation

## Quality Standards

1. **Source citation** — Every piece of advice must link to its authoritative source
2. **Actionability** — Guidance should include concrete steps, not abstract principles
3. **Threat severity** — Mark critical notices (supply chain attacks) with ⚠️ CRITICAL headers
4. **Freshness** — Include "Last updated: YYYY-MM-DD" on time-sensitive content
5. **Cross-referencing** — Link between related topics (e.g., vulnerability management → container scanning)

## Workflow

### Adding New Guidance

1. **Source it** — Find the authoritative source (Chainguard Academy, NIST, CVE database, etc.)
2. **Summarize** — Capture the key points in your own words
3. **Link** — Always include URL to the original
4. **Organize** — Place in appropriate directory
5. **Commit** — `docs: add [topic] from [source]`

### Updating Threat Intelligence

When a new security threat emerges:
1. Create a file in `security-advisories/` with the threat name
2. Document: what, when, affected versions, mitigation steps, sources
3. Mark with ⚠️ CRITICAL if it requires immediate action
4. Cross-link from affected projects (e.g., add a "See also" in Clearwatch's CLAUDE.md)

### No Merges Without Cross-Project Check

Before committing changes that might affect other projects (e.g., new container hardening requirements):
1. Check if Clearwatch, infrastructure, or homelab reference this guidance
2. Verify they won't be broken by the change
3. Note in commit message if this updates guidance that other projects depend on

## Git Discipline

- Commit message format: `docs: [add|update|clarify] [topic]`
- Include source URL in commit message if adding new guidance
- Push to origin master immediately after commit
- No uncommitted changes should accumulate

## Integration with Other Projects

**Clearwatch** (`~/projects/clearwatch/`):
- References container security guidance
- Links to Chainguard image best practices

**Infrastructure** (`~/projects/infrastructure/` or k8s deployment):
- Uses container hardening framework
- References policy enforcement patterns

**Homelab** (`~/projects/homelab/`):
- Uses vulnerability scanning guidance
- References update strategies

When updating guidance that affects these projects, ensure they can still reference it correctly.

## No Code Here

This is **documentation and guidance only**. If you need to:
- Write scripts for container scanning → homelab project
- Implement policy enforcement → infrastructure project
- Generate security reports → Clearwatch project

Keep implementation code in the project that uses it, with documentation references to `advice/`.

---

**Last updated: 2026-03-25**
