# Chainguard: Distroless Image Patterns

**Source:** Chainguard Academy, Container Security Best Practices
**Last updated:** 2026-03-25
**Scope:** How to build and use distroless images effectively

## What Is Distroless?

Distroless images are container images with **only the application + minimal runtime**. They exclude:
- ❌ Package managers (apt, yum, apk)
- ❌ Shells (/bin/sh, /bin/bash)
- ❌ Standard utilities (grep, sed, find, curl, etc.)
- ❌ Debuggers and development tools
- ❌ Non-root user accounts (typically)

**Result:** An image that is 30-70% smaller and has 90%+ fewer CVEs than full-OS images.

## Why Distroless

### Security Benefits
1. **Smaller attack surface** — No tools to exploit, no package manager vulnerabilities
2. **No shell escape** — Attacker can't access /bin/sh to spawn interactive shell
3. **CVE reduction** — Fewer packages = fewer CVEs
   - Ubuntu 22.04: 50+ CVEs at pull
   - Chainguard distroless: 0-2 CVEs at pull
4. **Immutable application** — Can't install new packages post-deployment
5. **Compliance advantage** — Simpler to audit what's in the image

### Operational Benefits
1. **Faster pulls** — Smaller images = faster deployment
2. **Lower storage cost** — ~20MB vs. 77MB for Ubuntu
3. **Cleaner supply chain** — Know exactly what's in the image
4. **Reproducible builds** — No "OS updates" changing what ships

## The Trade-off

**Can't debug in production.** Distroless images have no shell, so:
- ❌ Can't `docker exec -it <container> /bin/bash`
- ❌ Can't install tools with package manager
- ❌ Can't inspect logs with grep/tail directly
- ❌ Can't network debug with curl/netcat

**Mitigation:** Debug images (see "Debug Patterns" below)

## Building Distroless Images

### Pattern 1: Multi-Stage Build (Recommended)

```dockerfile
# Build stage: full dev environment
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp .

# Runtime stage: distroless
FROM cgr.dev/chainguard/static:latest
COPY --from=builder /app/myapp /app
USER 65532:65532
ENTRYPOINT ["/app"]
```

**Result:**
- Build artifacts (go cache, source code) not in final image
- Final image: only the binary + base distroless
- Size: ~20MB vs. ~400MB with full Go image

### Pattern 2: Language-Specific Distroless

Chainguard provides distroless images for common languages:

```dockerfile
# Python app
FROM cgr.dev/chainguard/python:latest as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt
COPY . .

FROM cgr.dev/chainguard/python:latest
COPY --from=builder /root/.local /root/.local
COPY --from=builder /app /app
ENV PATH=/root/.local/bin:$PATH
WORKDIR /app
USER 65532:65532
ENTRYPOINT ["python", "app.py"]
```

**Chainguard distroless images available:**
- `cgr.dev/chainguard/static` — For statically-linked binaries (Go, Rust)
- `cgr.dev/chainguard/python` — With Python runtime
- `cgr.dev/chainguard/nodejs` — With Node.js runtime
- `cgr.dev/chainguard/go` — Go with build tools (for multi-stage)
- `cgr.dev/chainguard/base` — Minimal OS utilities (rarely needed)

### Pattern 3: Alpine as Intermediate (Acceptable Compromise)

If distroless is too strict, Alpine is a reasonable middle ground:

```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp .

# Alpine: ~10MB, has apk but no large standard library
FROM alpine:latest
RUN apk add --no-cache ca-certificates
COPY --from=builder /app/myapp /app
USER 1000:1000
ENTRYPOINT ["/app"]
```

**Trade-off:**
- ✅ Still much smaller than Ubuntu (10MB vs. 77MB)
- ✅ Has package manager for emergencies
- ❌ More CVEs than distroless (~5-10 vs. 0-2)
- ❌ Has /bin/sh, slightly higher attack surface

**When to use:** If you need package manager access OR need some utilities for initialization.

## Distroless Challenges & Solutions

### Challenge 1: "How do I debug?"

**Solution: Debug Images**

Chainguard provides `-debug` variants with shell access:

```bash
# Production: distroless
FROM cgr.dev/chainguard/python:latest

# Development/debug: -debug variant
FROM cgr.dev/chainguard/python:latest-debug
```

Run with:
```bash
# Debug version (has /bin/sh)
docker run --rm -it cgr.dev/chainguard/python:latest-debug /bin/sh

# Or use in CI for diagnostics
docker run --rm cgr.dev/chainguard/python:latest-debug sh -c "python -m pip list"
```

**Use case:** Debug images in dev, test distroless in CI/production.

### Challenge 2: "I need curl or wget"

**Solution: Use sidecar or init container**

Instead of bundling curl in the application image:

```yaml
# Kubernetes example
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  initContainers:
  - name: check-dependency
    image: cgr.dev/chainguard/curl:latest
    command: ["curl", "http://dependency:8080/health"]
  containers:
  - name: app
    image: myapp:latest  # distroless
```

**Or in Docker Compose:**
```yaml
services:
  check:
    image: cgr.dev/chainguard/curl:latest
    command: curl http://dependency:8080
    depends_on:
      - dependency
  app:
    image: myapp:latest  # distroless
    depends_on:
      - check
```

### Challenge 3: "I need to read config files at runtime"

**Solution: Config injection patterns**

Don't rely on `sed` to modify configs at runtime. Instead:

**Option A: Build-time configuration**
```dockerfile
ARG ENV=prod
COPY config.${ENV}.yaml /etc/myapp/config.yaml
```

**Option B: Environment variables**
```dockerfile
# Application reads from env vars
ENV DB_HOST=localhost
ENV DB_PORT=5432
```

**Option C: Volume mount (Kubernetes)**
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /etc/myapp
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Challenge 4: "I can't run `pip install` at startup"

**Solution: Bake everything into the image**

```dockerfile
FROM cgr.dev/chainguard/python:latest as builder
WORKDIR /app
COPY requirements.txt .
# Install at build time, not runtime
RUN pip install --user -r requirements.txt

FROM cgr.dev/chainguard/python:latest
COPY --from=builder /root/.local /root/.local
COPY --from=builder /app /app
ENV PATH=/root/.local/bin:$PATH
ENTRYPOINT ["python", "app.py"]
```

**Principle:** Distroless means the image is static. All dependencies must be baked in at build time.

## Verification Checklist

```bash
# 1. Verify distroless (no /bin/sh)
docker run --rm myimage /bin/sh
# Expected: error "exec /bin/sh: no such file or directory"

# 2. Verify minimal contents
docker inspect myimage | grep -i cmd
# Expected: only the application entrypoint

# 3. Scan for CVEs
grype myimage
# Expected: 0-2 CVEs (not 50+)

# 4. Verify non-root user
docker run --rm --entrypoint id myimage
# Expected: uid=65532, gid=65532 (or similar)

# 5. Verify read-only root (if deployed with securityContext)
docker run --rm --read-only --tmpfs /tmp myimage
# Expected: no "read-only filesystem" errors on startup
```

## When Distroless Is NOT Appropriate

- ❌ Development images (use `-debug` variants instead)
- ❌ Applications requiring shell interaction at startup
- ❌ Legacy applications that can't be refactored to bake in config
- ❌ Images where you genuinely need runtime tools for compliance (rare)

**For these cases:** Use Alpine or `-debug` variants, but never full-OS in production.

## References

- **Chainguard Distroless Images:** https://www.chainguard.dev/chainguard-images
- **How Chainguard Creates Low-CVE Images:** https://edu.chainguard.dev/chainguard/chainguard-images/about/zerocve/
- **Google Distroless (alternative):** https://github.com/GoogleContainerTools/distroless
- **Multi-stage Dockerfile best practices:** https://docs.docker.com/build/building/multi-stage/
