# ⚠️ CRITICAL: LiteLLM Supply Chain Attack (PyPI Compromise)

**Last updated: 2026-03-25**
**Severity: P0 (Complete system compromise)**
**Status: Active threat**

## Summary

Malicious versions of LiteLLM (versions 1.82.7 and 1.82.8) were uploaded to PyPI after a maintainer account was compromised. These versions contain credential-stealing malware that exfiltrates AWS keys, GCP credentials, GitHub tokens, SSH private keys, and cryptocurrency wallet keys.

## Attack Timeline

- **Breach:** Maintainer account compromised (attributed to TeamPCP)
- **Malicious versions published:** 1.82.7, 1.82.8 on PyPI
- **Last clean version:** 1.82.6
- **Status:** Versions removed from PyPI, but anyone who installed 1.82.7 or 1.82.8 is compromised

## What the Malware Does

If 1.82.7 or 1.82.8 is installed, the infostealer immediately exfiltrates:
- AWS access keys and secrets
- GCP service account credentials
- GitHub personal access tokens
- SSH private keys (`~/.ssh/`)
- Cryptocurrency wallet files and keys
- Any other credentials in standard locations

**Attack surface:** Complete. Assume total compromise of all credentials on the system.

## Immediate Actions Required

### 1. Check Installation Status

```bash
pip list | grep litellm
pip show litellm  # Check version
```

### 2. If Installed (Any Version)

**Do NOT update LiteLLM.** Verify version:

- ✅ **SAFE:** 1.82.6 or earlier
- ❌ **COMPROMISED:** 1.82.7, 1.82.8 or later
- ⚠️ **UNKNOWN:** Newer versions — check Chainguard advisories before updating

### 3. If Versions 1.82.7 or 1.82.8 Were Ever Installed

Your system is compromised. Required steps:

1. **Uninstall immediately:** `pip uninstall litellm`
2. **Rotate ALL credentials:**
   - AWS access keys and secrets
   - GCP service accounts and keys
   - GitHub personal access tokens
   - SSH private keys (generate new ones)
   - Any API tokens used by the system
   - Crypto wallet keys (consider funds lost)
3. **Check for exfiltration indicators:**
   - Look for `litellm_init.pth` file (indicator of malware execution)
   - Check AWS CloudTrail for unauthorized access
   - Review GCP Cloud Audit Logs
   - Monitor Git push logs on GitHub
4. **Treat machine as fully compromised** — all passwords, secrets, and credentials should be considered known to attackers

## Mitigation for Future

### Pin LiteLLM to Safe Version

If your project uses LiteLLM, pin to 1.82.6 or explicitly exclude dangerous versions:

**requirements.txt:**
```
litellm==1.82.6
```

**pyproject.toml:**
```toml
[project]
dependencies = [
    "litellm==1.82.6",  # DO NOT UPDATE without explicit review
]
```

**pip constraints file:**
```
litellm==1.82.6
```

### Block Auto-Updates

If using automated dependency update tools (Renovate, Dependabot):
1. **Exclude LiteLLM from automatic updates** — require manual review and testing
2. **Set maximum version constraint:** `litellm<1.82.7` to prevent accidental upgrades
3. **Monitor Chainguard advisories** before allowing any update

### Monitor for Malicious Versions

- [Chainguard LiteLLM Vulnerabilities Page](https://images.chainguard.dev/directory/image/litellm/vulnerabilities)
- [GitHub Security Advisory Database](https://github.com/advisories/GHSA-53gh-p8jc-7rg8)
- [LiteLLM GitHub Repository](https://github.com/BerriAI/litellm) — watch for security announcements

## Why "DO NOT UPDATE"

Updating LiteLLM without explicit verification risks re-introducing the malware or moving to a new compromised version. The maintainer account is compromised, so future releases are not automatically trustworthy until the PyPI account is fully secured and verified.

## References

- [OX Security: LiteLLM PyPI Malware Analysis](https://www.ox.security/blog/litellm-malware-malicious-pypi-versions-steal-cloud-and-crypto-credentials/)
- [GitHub Advisory: CVE-2024-6825 (LiteLLM RCE)](https://github.com/advisories/GHSA-53gh-p8jc-7rg8)
- [Chainguard LiteLLM Vulnerabilities](https://images.chainguard.dev/directory/image/litellm/vulnerabilities)
- [XDA Developers: Popular Python Library Backdoor](https://www.xda-developers.com/popular-python-library-backdoor-machine/)
- [CyberInsider: Supply Chain Attack](https://cyberinsider.com/new-supply-chain-attack-hits-litellm-with-95m-monthly-downloads/)

## Related Guidance

- See `advice/container-security/dependency-vetting.md` for supply chain security practices
- See `advice/supply-chain/` for other similar threats

---

**If you have installed 1.82.7 or 1.82.8, do not delay — rotate credentials immediately.**
