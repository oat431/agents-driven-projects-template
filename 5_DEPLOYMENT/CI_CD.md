# CI_CD.md — Pipeline Design & Automation

<!--
  The CI/CD pipeline. AI agents use this to:
  - Write correct pipeline configs (GitHub Actions, GitLab CI)
  - Understand what runs when
  - Debug pipeline failures
-->

## Pipeline Overview

```
PUSH → BUILD → LINT → UNIT TEST → INTEGRATION TEST → BUILD IMAGE → DEPLOY
                                                          │
                                                          ▼
                                                    SECURITY SCAN
                                                          │
                                                          ▼
                                                    PUSH TO REGISTRY
```

---

## <!-- GitHub Actions Example -->

### Pipeline Triggers

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [develop, 'feature/**']
  pull_request:
    branches: [develop, main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
        ports: ['5432:5432']
      redis:
        image: redis:7
        ports: ['6379:6379']

    steps:
      - uses: actions/checkout@v4

      # --- BUILD ---
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Build
        run: ./mvnw clean compile

      # --- LINT ---
      - name: Checkstyle
        run: ./mvnw checkstyle:check

      - name: PMD
        run: ./mvnw pmd:check

      # --- UNIT TEST ---
      - name: Unit Tests
        run: ./mvnw test

      # --- INTEGRATION TEST ---
      - name: Integration Tests
        run: ./mvnw verify -P integration
        env:
          DATABASE_URL: jdbc:postgresql://localhost:5432/postgres
          DATABASE_USER: postgres
          DATABASE_PASSWORD: postgres
          REDIS_HOST: localhost

      # --- COVERAGE ---
      - name: Coverage Report
        run: ./mvnw jacoco:report

      - name: Check Coverage Thresholds
        run: |
          COVERAGE=$(jq '.counter[] | select(.type=="LINE") | .covered / (.covered + .missed) * 100' target/site/jacoco/jacoco.xml)
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% below 80% threshold"
            exit 1
          fi
```

### Docker Build + Push (on merge to main)

```yaml
# .github/workflows/docker-publish.yml
name: Docker Publish

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract version
        id: version
        run: |
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            echo "tag=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
          else
            echo "tag=latest" >> $GITHUB_OUTPUT
          fi

      - name: Build image
        run: docker build -t ghcr.io/panomete/auth-service:${{ steps.version.outputs.tag }} .

      - name: Scan for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/panomete/auth-service:${{ steps.version.outputs.tag }}
          format: 'table'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Push to registry
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u $GITHUB_ACTOR --password-stdin
          docker push ghcr.io/panomete/auth-service:${{ steps.version.outputs.tag }}
```

### Deploy (per environment)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, qa, uat, staging, prod]
        required: true
      version:
        type: string
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Deploy to ${{ inputs.environment }}
        run: |
          kubectl set image deployment/api-server \
            api-server=ghcr.io/panomete/auth-service:${{ inputs.version }} \
            --namespace=${{ inputs.environment }}

      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/api-server \
            --namespace=${{ inputs.environment }} \
            --timeout=5m

      - name: Smoke test
        run: |
          curl -s https://${{ inputs.environment }}.api.example.com/actuator/health | jq .status
```

---

## Pipeline Decision Matrix

| Event | Build | Lint | Unit Test | Integration | Security Scan | Deploy |
|-------|-------|------|-----------|-------------|---------------|--------|
| Push to feature branch | ✅ | ✅ | ✅ | — | — | — |
| PR to develop | ✅ | ✅ | ✅ | ✅ | — | — |
| Merge to develop | ✅ | ✅ | ✅ | ✅ | ✅ | → DEV (auto) |
| Promote to QA | — | — | — | ✅ (full) | ✅ | → QA (manual) |
| Promote to UAT | — | — | — | — | — | → UAT (manual) |
| Merge to main | ✅ | ✅ | ✅ | ✅ (full) | ✅ | → STAGING (manual) |
| Release tag | — | — | — | — | — | → PROD (restricted) |

---

## Required Secrets (GitHub Actions)

| Secret | Purpose | Environment |
|--------|---------|-------------|
| `DOCKER_REGISTRY_TOKEN` | Push images | All |
| `KUBE_CONFIG_DEV` | Deploy to dev | DEV |
| `KUBE_CONFIG_PROD` | Deploy to prod | PROD (restricted) |
| `SONAR_TOKEN` | Code quality | All |
| `SNYK_TOKEN` | Dependency scan | All |

Never store production secrets in repo-level variables. Use environment-specific secrets with approval gates.

---

## Pipeline Time Targets

| Stage | Target | If Slower... |
|-------|--------|-------------|
| Build | <2 min | Check dependency cache |
| Unit tests | <30 sec | Parallelize test classes |
| Integration tests | <3 min | Use TestContainers reuse mode |
| Docker build | <2 min | Multi-stage build, layer caching |
| Full pipeline | <10 min | Parallelize where possible |

---

## Rules for AI Agents
1. **Never commit secrets** to pipeline files. Use GitHub Secrets / Vault.
2. **Pipeline code is production code.** Review it. Test it.
3. **New service = new pipeline job.** Don't bloat the main pipeline.
4. **Failed pipeline = no deploy.** Fix before promoting.
5. **Security scan failure = BLOCK deploy.** Fix the vulnerability.
