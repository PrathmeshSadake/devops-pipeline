# GitHub Actions CI/CD Pipeline Guide
## From Beginner to Production-Grade

---

## Table of Contents
1. [Introduction](#introduction)
2. [CI/CD Pipeline Flow](#pipeline-flow)
3. [Core Concepts](#core-concepts)
4. [Beginner Level Examples](#beginner-examples)
5. [Intermediate Level Examples](#intermediate-examples)
6. [Advanced Production-Grade Examples](#advanced-examples)
7. [Best Practices](#best-practices)
8. [Quick Reference](#quick-reference)
9. [Common Patterns](#common-patterns)

---

## Introduction

GitHub Actions is a CI/CD platform that allows you to automate your build, test, and deployment pipeline. Workflows are defined in YAML files stored in `.github/workflows/` directory.

### Key Benefits
- **Free for public repositories**
- **Integrated with GitHub**
- **Marketplace with 13,000+ actions**
- **Matrix builds for parallel testing**
- **Self-hosted runners support**

---

## CI/CD Pipeline Flow

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐      ┌──────────┐
│ Code Push   │ ───> │ Build & Test │ ───> │  Package    │ ───> │  Deploy  │
│  (Trigger)  │      │  (CI Phase)  │      │ (Artifacts) │      │ (CD Phase)│
└─────────────┘      └──────────────┘      └─────────────┘      └──────────┘
```

**Stages Explained:**
1. **Trigger**: Code push, PR, schedule, or manual trigger
2. **Build & Test**: Compile code, run tests, lint, security scans
3. **Package**: Create Docker images, build artifacts, generate releases
4. **Deploy**: Deploy to staging/production environments

---

## Core Concepts

### 1. Workflow
A configurable automated process defined in a YAML file.

```yaml
name: My Workflow
on: [push]
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello World"
```

### 2. Events (Triggers)
Events that trigger workflow execution.

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:  # Manual trigger
```

### 3. Jobs
A set of steps that execute on the same runner.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
  
  test:
    needs: build  # Runs after build completes
    runs-on: ubuntu-latest
    steps:
      - run: npm test
```

### 4. Steps
Individual tasks within a job.

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4
  
  - name: Run a command
    run: echo "Hello World"
  
  - name: Multi-line script
    run: |
      echo "Line 1"
      echo "Line 2"
```

### 5. Actions
Reusable units of code.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'
```

### 6. Runners
Servers that run your workflows.

```yaml
runs-on: ubuntu-latest      # GitHub-hosted
runs-on: windows-latest
runs-on: macos-latest
runs-on: self-hosted        # Your own server
```

### 7. Matrix Strategy
Run jobs across multiple configurations.

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest]
```

### 8. Artifacts
Files shared between jobs or preserved after completion.

```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: dist/

- name: Download artifact
  uses: actions/download-artifact@v4
  with:
    name: my-artifact
```

### 9. Secrets and Environment Variables

```yaml
env:
  NODE_ENV: production
  
steps:
  - name: Use secret
    run: echo "Secret is ${{ secrets.MY_SECRET }}"
    env:
      API_KEY: ${{ secrets.API_KEY }}
```

### 10. Environments
Deployment targets with protection rules.

```yaml
jobs:
  deploy:
    environment:
      name: production
      url: https://myapp.com
    steps:
      - run: deploy.sh
```

---

## Beginner Examples

### Example 1: Basic CI Pipeline

**File**: `.github/workflows/basic-ci.yml`

```yaml
name: Basic CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Build application
      run: npm run build
```

**What it does:**
- Triggers on push to main/develop or PRs to main
- Checks out code
- Sets up Node.js 18
- Installs dependencies with `npm ci` (faster, cleaner than `npm install`)
- Runs tests
- Builds the application

---

### Example 2: Python Application CI

**File**: `.github/workflows/python-ci.yml`

```yaml
name: Python CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest pytest-cov
    
    - name: Run tests with coverage
      run: |
        pytest --cov=src tests/
    
    - name: Lint with flake8
      run: |
        pip install flake8
        flake8 src/ --count --max-line-length=127
```

---

### Example 3: Docker Build

**File**: `.github/workflows/docker-build.yml`

```yaml
name: Docker Build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build Docker image
      run: docker build -t myapp:latest .
    
    - name: Test Docker image
      run: |
        docker run --rm myapp:latest echo "Testing image"
```

---

## Intermediate Examples

### Example 4: Multi-Environment Pipeline with Matrix Testing

**File**: `.github/workflows/multi-env.yml`

```yaml
name: Multi-Environment CI/CD

on:
  push:
    branches: [ main, staging, develop ]
  pull_request:
    branches: [ main, staging ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io

jobs:
  # Run tests across multiple Node versions
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run unit tests
      run: npm test -- --coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage/lcov.info
        flags: node-${{ matrix.node-version }}

  # Build application
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install and Build
      run: |
        npm ci
        npm run build
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: build-artifacts
        path: dist/
        retention-days: 7

  # Deploy to staging
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    environment: 
      name: staging
      url: https://staging.myapp.com
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: build-artifacts
        path: dist/
    
    - name: Deploy to staging server
      run: |
        echo "Deploying to staging..."
        # Add your deployment commands here
        # rsync -avz dist/ user@staging-server:/var/www/
    
    - name: Run smoke tests
      run: |
        curl -f https://staging.myapp.com/health || exit 1

  # Deploy to production
  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://myapp.com
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: build-artifacts
        path: dist/
    
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Add your deployment commands here
    
    - name: Health check
      run: |
        sleep 10
        curl -f https://myapp.com/health || exit 1
```

**Key Features:**
- **Matrix testing**: Tests across Node 16, 18, and 20
- **Artifacts**: Build once, deploy multiple times
- **Conditional deployment**: Different branches deploy to different environments
- **Environment protection**: Uses GitHub environments with URLs
- **Health checks**: Validates deployment success

---

### Example 5: Monorepo with Changed Files Detection

**File**: `.github/workflows/monorepo.yml`

```yaml
name: Monorepo CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      backend: ${{ steps.changes.outputs.backend }}
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: dorny/paths-filter@v2
      id: changes
      with:
        filters: |
          frontend:
            - 'frontend/**'
          backend:
            - 'backend/**'

  test-frontend:
    needs: detect-changes
    if: needs.detect-changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    - name: Test frontend
      run: |
        cd frontend
        npm ci
        npm test

  test-backend:
    needs: detect-changes
    if: needs.detect-changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    - name: Test backend
      run: |
        cd backend
        pip install -r requirements.txt
        pytest
```

---

## Advanced Production-Grade Examples

### Example 6: Full Production Pipeline with Docker

**File**: `.github/workflows/production.yml`

```yaml
name: Production CI/CD

on:
  push:
    branches: [ main, staging ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Code quality checks
  code-quality:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0  # Full history for better analysis
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run ESLint
      run: npm run lint -- --format json --output-file eslint-report.json
      continue-on-error: true
    
    - name: SonarCloud Scan
      uses: SonarSource/sonarcloud-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      with:
        args: >
          -Dsonar.projectKey=my-project
          -Dsonar.organization=my-org
    
    - name: Security audit
      run: npm audit --audit-level=high
    
    - name: Check for outdated dependencies
      run: npm outdated || true

  # Run comprehensive tests
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node-version: [18, 20]
        test-suite: [unit, integration]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run ${{ matrix.test-suite }} tests
      run: npm run test:${{ matrix.test-suite }} -- --coverage --maxWorkers=2
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./coverage/lcov.info
        flags: ${{ matrix.test-suite }}-node${{ matrix.node-version }}
        name: codecov-${{ matrix.node-version }}-${{ matrix.test-suite }}
        fail_ci_if_error: false

  # Build and push Docker image
  build-and-push:
    needs: [code-quality, test]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}
    
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        push: ${{ github.event_name != 'pull_request' }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        platforms: linux/amd64,linux/arm64
        build-args: |
          BUILD_DATE=${{ github.event.head_commit.timestamp }}
          VCS_REF=${{ github.sha }}
          VERSION=${{ steps.meta.outputs.version }}
    
    - name: Sign image with Cosign
      if: github.event_name != 'pull_request'
      run: |
        cosign sign --yes ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}

  # Deploy to staging environment
  deploy-staging:
    needs: build-and-push
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Update ECS task definition
      run: |
        aws ecs describe-task-definition \
          --task-definition staging-app \
          --query taskDefinition > task-def.json
        
        jq '.containerDefinitions[0].image = "${{ needs.build-and-push.outputs.image-tag }}"' \
          task-def.json > new-task-def.json
    
    - name: Deploy to ECS
      run: |
        aws ecs register-task-definition \
          --cli-input-json file://new-task-def.json
        
        aws ecs update-service \
          --cluster staging-cluster \
          --service app-service \
          --task-definition staging-app \
          --force-new-deployment
    
    - name: Wait for service stability
      run: |
        aws ecs wait services-stable \
          --cluster staging-cluster \
          --services app-service
    
    - name: Run smoke tests
      run: |
        npm ci
        npm run test:smoke -- --env staging
    
    - name: Performance tests
      run: |
        npm run test:performance -- --env staging

  # Deploy to production with blue-green deployment
  deploy-production:
    needs: build-and-push
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
        aws-region: us-east-1
    
    - name: Deploy with Blue/Green
      run: |
        # Create new target group
        aws elbv2 create-target-group \
          --name prod-app-green \
          --protocol HTTP \
          --port 80 \
          --vpc-id ${{ secrets.VPC_ID }}
        
        # Update ECS service with new task definition
        aws ecs update-service \
          --cluster prod-cluster \
          --service app-service \
          --task-definition prod-app:${{ github.run_number }} \
          --force-new-deployment
    
    - name: Wait for deployment
      run: |
        aws ecs wait services-stable \
          --cluster prod-cluster \
          --services app-service
    
    - name: Run smoke tests on production
      run: |
        npm ci
        npm run test:smoke -- --env production --critical-only
      timeout-minutes: 5
    
    - name: Switch traffic to green
      if: success()
      run: |
        aws elbv2 modify-listener \
          --listener-arn ${{ secrets.LISTENER_ARN }} \
          --default-actions Type=forward,TargetGroupArn=$GREEN_TG_ARN
    
    - name: Rollback on failure
      if: failure()
      run: |
        aws ecs update-service \
          --cluster prod-cluster \
          --service app-service \
          --task-definition prod-app:previous \
          --force-new-deployment
    
    - name: Create GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        body: |
          ## Changes in this Release
          ${{ github.event.head_commit.message }}
          
          **Docker Image:** ${{ needs.build-and-push.outputs.image-tag }}
          **Digest:** ${{ needs.build-and-push.outputs.image-digest }}
        files: |
          dist/**/*
          CHANGELOG.md
    
    - name: Update deployment tracking
      run: |
        curl -X POST ${{ secrets.DEPLOYMENT_TRACKER_URL }} \
          -H "Content-Type: application/json" \
          -d '{
            "version": "${{ github.ref_name }}",
            "environment": "production",
            "status": "success",
            "timestamp": "${{ github.event.head_commit.timestamp }}"
          }'

  # Notify stakeholders
  notify:
    needs: [deploy-staging, deploy-production]
    if: always()
    runs-on: ubuntu-latest
    
    steps:
    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: |
          Deployment completed!
          Environment: ${{ github.ref_name }}
          Status: ${{ job.status }}
          Commit: ${{ github.sha }}
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      if: always()
    
    - name: Send email notification
      if: failure()
      uses: dawidd6/action-send-mail@v3
      with:
        server_address: smtp.gmail.com
        server_port: 465
        username: ${{ secrets.EMAIL_USERNAME }}
        password: ${{ secrets.EMAIL_PASSWORD }}
        subject: "Deployment Failed: ${{ github.repository }}"
        body: |
          Deployment failed for ${{ github.ref_name }}
          Workflow: ${{ github.workflow }}
          Run: ${{ github.run_id }}
        to: devops@mycompany.com
```

**Advanced Features:**
- **Comprehensive testing**: Unit, integration, and E2E tests
- **Code quality**: SonarCloud, ESLint, security audits
- **Multi-platform Docker**: Builds for AMD64 and ARM64
- **Image signing**: Uses Cosign for container signing
- **Blue-Green deployment**: Zero-downtime deployments
- **Smoke tests**: Validates deployment health
- **Automatic rollback**: Reverts on failure
- **Release automation**: Creates GitHub releases
- **Multi-channel notifications**: Slack and email

---

## Best Practices

### Security Best Practices

#### 1. Use Secrets, Never Hardcode
```yaml
# ❌ Bad
env:
  API_KEY: "abc123secret"

# ✅ Good
env:
  API_KEY: ${{ secrets.API_KEY }}
```

#### 2. Pin Action Versions
```yaml
# ❌ Bad (can break unexpectedly)
- uses: actions/checkout@v4

# ✅ Better (specific version)
- uses: actions/checkout@v4.1.0

# ✅ Best for production (full SHA)
- uses: actions/checkout@8e5e7e5ab8b370d6c329ec480221332ada57f0ab
```

#### 3. Minimal Permissions
```yaml
permissions:
  contents: read       # Read repo contents
  packages: write      # Write to GitHub Packages
  pull-requests: write # Comment on PRs
  # Don't grant permissions you don't need
```

#### 4. Secure Secrets Management
```yaml
# Use environment-specific secrets
jobs:
  deploy:
    environment: production  # Requires approval
    steps:
      - run: deploy.sh
        env:
          SECRET: ${{ secrets.PROD_SECRET }}
```

#### 5. Scan for Vulnerabilities
```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:latest
    format: 'sarif'
    output: 'trivy-results.sarif'

- name: Upload Trivy results to GitHub Security
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

---

### Performance Best Practices

#### 1. Cache Dependencies
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'  # Automatically caches npm dependencies

# Or manually cache
- uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

#### 2. Use Matrix for Parallel Testing
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest, macos-latest]
# Runs 9 jobs in parallel (3 versions × 3 OSes)
```

#### 3. Docker Layer Caching
```yaml
- name: Build Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

#### 4. Optimize Checkout
```yaml
# Only fetch what you need
- uses: actions/checkout@v4
  with:
    fetch-depth: 1  # Shallow clone
    sparse-checkout: |  # Only checkout specific paths
      src/
      tests/
```

#### 5. Set Timeouts
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Prevent hanging jobs
    
    steps:
    - name: Run tests
      run: npm test
      timeout-minutes: 10  # Step-level timeout
```

---

### Reliability Best Practices

#### 1. Retry Flaky Steps
```yaml
- name: Flaky external API call
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 5
    max_attempts: 3
    command: npm run integration-test
```

#### 2. Continue on Error Strategically
```yaml
- name: Optional linting
  run: npm run lint
  continue-on-error: true  # Don't fail build on lint errors

- name: Critical security scan
  run: npm audit
  # No continue-on-error - should fail the build
```

#### 3. Conditional Execution
```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main' && success()
  run: ./deploy.sh

- name: Notify on failure
  if: failure()
  run: ./notify-failure.sh
```

#### 4. Health Checks
```yaml
- name: Deploy application
  run: ./deploy.sh

- name: Wait for service
  run: sleep 30

- name: Health check
  run: |
    curl -f https://myapp.com/health || exit 1
```

#### 5. Status Checks
```yaml
# In GitHub repository settings, require these checks to pass:
# - test (all matrix combinations)
# - code-quality
# - security-scan
```

---

### Maintainability Best Practices

#### 1. Use Reusable Workflows
**File**: `.github/workflows/reusable-build.yml`
```yaml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      NPM_TOKEN:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run build
```

**Usage**:
```yaml
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '18'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### 2. Create Composite Actions
**File**: `.github/actions/setup-app/action.yml`
```yaml
name: 'Setup Application'
description: 'Setup Node.js and install dependencies'

inputs:
  node-version:
    description: 'Node.js version'
    required: true
    default: '18'

runs:
  using: 'composite'
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
    
    - run: npm ci
      shell: bash
```

**Usage**:
```yaml
- uses: ./.github/actions/setup-app
  with:
    node-version: '18'
```

#### 3. Document Workflows
```yaml
name: Production Deployment

# Purpose: Deploys application to production on tag push
# Requires: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY secrets
# Environments: production (with manual approval)
# Notifications: Slack #deployments channel

on:
  push:
    tags: [ 'v*' ]
```

#### 4. Use Environment Files
```yaml
- name: Set environment variables
  run: |
    echo "DATABASE_URL=${{ secrets.DATABASE_URL }}" >> $GITHUB_ENV
    echo "API_KEY=${{ secrets.API_KEY }}" >> $GITHUB_ENV

- name: Use environment variables
  run: |
    echo "Database: $DATABASE_URL"
    echo "API Key: $API_KEY"
```

---

## Quick Reference

### Common Workflow Triggers

```yaml
on:
  # Single event
  push:
  
  # Multiple events
  [push, pull_request]
  
  # Event with filters
  push:
    branches:
      - main
      - 'releases/**'
    paths:
      - 'src/**'
      - '!src/docs/**'
    tags:
      - 'v*'
  
  # Pull request types
  pull_request:
    types: [opened, synchronize, reopened]
  
  # Scheduled (cron)
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
    - cron: '0 */6 * * *'  # Every 6 hours
  
  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
        type: string
  
  # Trigger from another workflow
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]
  
  # Release events
  release:
    types: [published, created]
  
  # Issue and PR comments
  issue_comment:
    types: [created]
```

---

### Context Variables

```yaml
# GitHub context
${{ github.actor }}              # User who triggered workflow
${{ github.event_name }}         # Event that triggered workflow
${{ github.ref }}                # Branch or tag ref
${{ github.ref_name }}           # Branch or tag name only
${{ github.sha }}                # Commit SHA
${{ github.repository }}         # owner/repo
${{ github.repository_owner }}   # Repository owner
${{ github.run_id }}             # Unique workflow run ID
${{ github.run_number }}         # Run number for this workflow

# Job context
${{ job.status }}                # Current job status

# Steps context
${{ steps.step-id.outputs.result }}  # Output from previous step

# Runner context
${{ runner.os }}                 # linux, windows, or macos
${{ runner.temp }}               # Temp directory path
${{ runner.tool_cache }}         # Tool cache directory

# Environment variables
${{ env.MY_VAR }}                # Custom environment variable
```

---

### Conditional Expressions

```yaml
# Simple conditions
if: success()                    # Previous steps succeeded
if: failure()                    # Previous steps failed
if: always()                     # Run regardless of previous steps
if: cancelled()                  # Workflow was cancelled

# Check event type
if: github.event_name == 'push'

# Check branch
if: github.ref == 'refs/heads/main'
if: startsWith(github.ref, 'refs/tags/')

# Multiple conditions (AND)
if: success() && github.ref == 'refs/heads/main'

# Multiple conditions (OR)
if: github.event_name == 'push' || github.event_name == 'pull_request'

# NOT condition
if: "!cancelled()"

# Check if secret exists
if: secrets.MY_SECRET != ''

# Complex condition
if: |
  (github.event_name == 'push' && github.ref == 'refs/heads/main') ||
  (github.event_name == 'pull_request' && github.base_ref == 'main')
```

---

### Popular Actions

```yaml
# Checkout code
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    submodules: recursive
    token: ${{ secrets.GITHUB_TOKEN }}

# Setup runtimes
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'

- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
    cache: 'pip'

- uses: actions/setup-java@v3
  with:
    distribution: 'temurin'
    java-version: '17'

- uses: actions/setup-go@v4
  with:
    go-version: '1.21'

# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# Upload/Download artifacts
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: dist/
    retention-days: 5

- uses: actions/download-artifact@v4
  with:
    name: my-artifact
    path: dist/

# Docker actions
- uses: docker/setup-buildx-action@v3

- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest

# Cloud providers
- uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- uses: google-github-actions/auth@v1
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}

# Create release
- uses: softprops/action-gh-release@v1
  with:
    files: |
      dist/**/*
      CHANGELOG.md
    body: Release notes here

# Slack notification
- uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Common Patterns

### Pattern 1: Build Once, Deploy Many

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  deploy-dev:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
      - run: deploy-to-dev.sh

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
      - run: deploy-to-staging.sh

  deploy-prod:
    needs: [build, deploy-staging]
    environment: production
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
      - run: deploy-to-prod.sh
```

---

### Pattern 2: Parallel Testing with Results Aggregation

```yaml
jobs:
  test:
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.shard }}
          path: coverage/

  combine-coverage:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          pattern: coverage-*
          merge-multiple: true
      - run: npm run coverage:merge
      - uses: codecov/codecov-action@v3
```

---

### Pattern 3: Canary Deployment

```yaml
jobs:
  deploy-canary:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy 10% traffic to new version
        run: |
          kubectl set image deployment/app app=myapp:${{ github.sha }}
          kubectl patch deployment app -p '{"spec":{"replicas":1}}'
      
      - name: Monitor metrics for 10 minutes
        run: |
          sleep 600
          ERROR_RATE=$(curl -s metrics-api/error-rate)
          if [ $ERROR_RATE -gt 5 ]; then
            echo "High error rate detected"
            exit 1
          fi
      
      - name: Gradually increase traffic
        run: |
          for i in 25 50 75 100; do
            kubectl scale deployment app --replicas=$((i/10))
            sleep 300
          done
```

---

### Pattern 4: Automated Dependency Updates

```yaml
name: Auto Update Dependencies

on:
  schedule:
    - cron: '0 0 * * 1'  # Every Monday
  workflow_dispatch:

jobs:
  update-deps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Update dependencies
        run: |
          npm update
          npm audit fix
      
      - name: Run tests
        run: npm test
      
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: 'chore: update dependencies'
          title: 'Automated dependency updates'
          body: |
            Automated dependency updates from weekly workflow.
            
            Please review the changes and merge if tests pass.
          branch: automated-deps-update
          labels: dependencies
```

---

### Pattern 5: Feature Flag Deployment

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy with feature flag disabled
        run: |
          kubectl set env deployment/app FEATURE_X_ENABLED=false
          kubectl rollout status deployment/app
      
      - name: Run smoke tests
        run: npm run test:smoke
      
      - name: Gradually enable feature flag
        run: |
          # Enable for 10% of users
          curl -X POST feature-flag-api/feature-x \
            -d '{"enabled": true, "percentage": 10}'
          sleep 300
          
          # Enable for 50% of users
          curl -X POST feature-flag-api/feature-x \
            -d '{"enabled": true, "percentage": 50}'
          sleep 300
          
          # Enable for 100% of users
          curl -X POST feature-flag-api/feature-x \
            -d '{"enabled": true, "percentage": 100}'
```

---

### Pattern 6: Database Migration

```yaml
jobs:
  migrate:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Backup database
        run: |
          pg_dump -h ${{ secrets.DB_HOST }} \
                  -U ${{ secrets.DB_USER }} \
                  ${{ secrets.DB_NAME }} \
          > backup_$(date +%Y%m%d_%H%M%S).sql
      
      - name: Upload backup
        uses: actions/upload-artifact@v4
        with:
          name: db-backup
          path: backup_*.sql
      
      - name: Run migrations
        run: npm run migrate
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
      
      - name: Verify migrations
        run: npm run migrate:verify
      
      - name: Rollback on failure
        if: failure()
        run: |
          npm run migrate:rollback
          echo "Migration failed and was rolled back"
```

---

### Pattern 7: Multi-Region Deployment

```yaml
jobs:
  deploy:
    strategy:
      matrix:
        region: [us-east-1, us-west-2, eu-west-1, ap-southeast-1]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ matrix.region }}
      
      - name: Deploy to ${{ matrix.region }}
        run: |
          aws ecs update-service \
            --cluster app-cluster-${{ matrix.region }} \
            --service app-service \
            --force-new-deployment
      
      - name: Wait for stability
        run: |
          aws ecs wait services-stable \
            --cluster app-cluster-${{ matrix.region }} \
            --services app-service
      
      - name: Health check
        run: |
          curl -f https://${{ matrix.region }}.myapp.com/health
```

---

### Pattern 8: Performance Testing in CI

```yaml
jobs:
  performance-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Start application
        run: |
          docker-compose up -d
          sleep 10
      
      - name: Run k6 performance tests
        uses: grafana/k6-action@v0.3.0
        with:
          filename: tests/performance.js
          flags: --out json=results.json
      
      - name: Parse results
        run: |
          P95=$(jq '.metrics.http_req_duration.values.p95' results.json)
          if (( $(echo "$P95 > 500" | bc -l) )); then
            echo "P95 latency ($P95ms) exceeds threshold (500ms)"
            exit 1
          fi
      
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: performance-results
          path: results.json
```

---

### Pattern 9: Preview Environments for PRs

```yaml
name: Preview Environment

on:
  pull_request:
    types: [opened, synchronize, closed]

jobs:
  deploy-preview:
    if: github.event.action != 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to preview environment
        run: |
          PREVIEW_URL="pr-${{ github.event.pull_request.number }}.preview.myapp.com"
          kubectl create namespace pr-${{ github.event.pull_request.number }}
          kubectl apply -f k8s/ -n pr-${{ github.event.pull_request.number }}
          kubectl set env deployment/app \
            BASE_URL=https://$PREVIEW_URL \
            -n pr-${{ github.event.pull_request.number }}
      
      - name: Comment PR with preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed to https://pr-${{ github.event.pull_request.number }}.preview.myapp.com'
            })
  
  cleanup-preview:
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - name: Delete preview environment
        run: |
          kubectl delete namespace pr-${{ github.event.pull_request.number }}
```

---

### Pattern 10: Semantic Versioning with Automated Releases

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Determine version bump
        id: semver
        run: |
          # Analyze commit messages
          if git log -1 --pretty=%B | grep -q "BREAKING CHANGE"; then
            echo "bump=major" >> $GITHUB_OUTPUT
          elif git log -1 --pretty=%B | grep -q "^feat"; then
            echo "bump=minor" >> $GITHUB_OUTPUT
          else
            echo "bump=patch" >> $GITHUB_OUTPUT
          fi
      
      - name: Bump version
        run: npm version ${{ steps.semver.outputs.bump }} -m "chore: release v%s"
      
      - name: Build and test
        run: |
          npm ci
          npm test
          npm run build
      
      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      
      - name: Push tags
        run: git push --follow-tags
      
      - name: Create GitHub release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: v${{ steps.semver.outputs.version }}
          generate_release_notes: true
```

---

## Troubleshooting Common Issues

### Issue 1: Workflow Not Triggering

**Problem**: Workflow doesn't run when expected

**Solutions**:
```yaml
# Check if path filters are too restrictive
on:
  push:
    paths:
      - 'src/**'  # Only triggers if files in src/ change
      - '!**.md'  # Excludes markdown files

# Verify branch names match exactly
on:
  push:
    branches:
      - main  # Must match exact branch name

# Check if workflow file is in correct location
# Must be in: .github/workflows/

# Ensure YAML is valid
# Use: yamllint .github/workflows/yourfile.yml
```

---

### Issue 2: Permission Denied Errors

**Problem**: `Permission denied` or `403 Forbidden` errors

**Solutions**:
```yaml
# Add necessary permissions
permissions:
  contents: write      # For pushing commits
  packages: write      # For pushing packages
  pull-requests: write # For commenting on PRs

# Use GITHUB_TOKEN correctly
- run: gh pr comment $PR_NUMBER --body "Comment"
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

# For GitHub App, use app token
- uses: actions/create-github-app-token@v1
  id: app-token
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}
```

---

### Issue 3: Secrets Not Available

**Problem**: Secrets are empty or undefined

**Solutions**:
```yaml
# Secrets aren't available in pull_request from forks
on:
  pull_request_target:  # Use this instead for forks
    types: [opened]

# Check secret is defined in correct scope
# Settings → Secrets → Actions

# Pass secrets explicitly to reusable workflows
jobs:
  call-workflow:
    uses: ./.github/workflows/reusable.yml
    secrets:
      MY_SECRET: ${{ secrets.MY_SECRET }}

# Verify secret names (case-sensitive)
env:
  API_KEY: ${{ secrets.API_KEY }}  # Must match exactly
```

---

### Issue 4: Caching Not Working

**Problem**: Dependencies reinstall every time

**Solutions**:
```yaml
# Use correct cache key
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-

# Use built-in caching where available
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'  # Automatic caching

# Clear cache if corrupted
# Settings → Actions → Caches → Delete specific cache
```

---

### Issue 5: Docker Build Failures

**Problem**: Docker builds fail or are slow

**Solutions**:
```yaml
# Use Docker layer caching
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max

# Build context issues
- uses: docker/build-push-action@v5
  with:
    context: .
    file: ./Dockerfile  # Specify Dockerfile location

# Multi-platform builds
- uses: docker/setup-qemu-action@v3  # Required for ARM builds
- uses: docker/setup-buildx-action@v3
- uses: docker/build-push-action@v5
  with:
    platforms: linux/amd64,linux/arm64
```

---

## Learning Resources

### Official Documentation
- **GitHub Actions Docs**: https://docs.github.com/actions
- **Workflow Syntax**: https://docs.github.com/actions/reference/workflow-syntax-for-github-actions
- **GitHub Actions Marketplace**: https://github.com/marketplace?type=actions

### Example Repositories
- **Starter Workflows**: https://github.com/actions/starter-workflows
- **GitHub Actions Examples**: https://github.com/sdras/awesome-actions

### Tools & Utilities
- **act**: Run GitHub Actions locally - https://github.com/nektos/act
- **actionlint**: Lint workflow files - https://github.com/rhysd/actionlint
- **GitHub CLI**: Interact with GitHub from terminal - https://cli.github.com/

### Community Resources
- **GitHub Community Forum**: https://github.community/
- **GitHub Actions on Reddit**: r/github
- **GitHub Actions Twitter**: @github

---

## Summary Checklist

### For Beginners ✅
- [ ] Understand triggers (on: push, pull_request)
- [ ] Create basic job with checkout and build steps
- [ ] Use secrets for sensitive data
- [ ] Set up basic CI for your language
- [ ] View workflow runs in Actions tab

### For Intermediate Users ✅
- [ ] Implement matrix strategy for parallel testing
- [ ] Use artifacts to share data between jobs
- [ ] Set up environment-specific deployments
- [ ] Configure caching for faster builds
- [ ] Add status badges to README

### For Advanced Users ✅
- [ ] Create reusable workflows and composite actions
- [ ] Implement blue-green or canary deployments
- [ ] Set up multi-region deployments
- [ ] Configure OIDC for cloud providers
- [ ] Monitor workflow performance and costs
- [ ] Implement automated security scanning
- [ ] Use self-hosted runners for specific needs

---

## Conclusion

GitHub Actions provides a powerful, flexible CI/CD platform that scales from simple automation to complex enterprise deployments. Key takeaways:

1. **Start Simple**: Begin with basic CI, gradually add complexity
2. **Security First**: Use secrets, pin versions, minimal permissions
3. **Optimize Performance**: Cache dependencies, use matrices, Docker layers
4. **Monitor & Iterate**: Track workflow performance, improve over time
5. **Reuse & Share**: Create reusable workflows and composite actions

Happy automating! 🚀