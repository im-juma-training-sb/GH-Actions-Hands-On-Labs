# Lab 2: Secrets and Environments - Secure CI/CD

**Duration:** 60-75 minutes  
**Level:** Beginner  
**Prerequisites:** Labs 1

---

## Overview

This lab teaches you how to securely manage sensitive information in GitHub Actions using secrets and environments. You'll create a realistic CI/CD pipeline that deploys to multiple environments (development, staging, production) with proper security controls and approval gates.

All exercises are performed directly in your GitHub repository.

---

## Exercise 1: Understanding Secrets (10 minutes)

### Learn why secrets are essential for secure workflows

**What are secrets?**
- Encrypted environment variables
- Never exposed in logs
- Used for API keys, tokens, passwords, certificates

**Where secrets live:**
1. **Repository secrets**: Available to all workflows in the repo
2. **Environment secrets**: Only available when deploying to specific environment
3. **Organization secrets**: Shared across multiple repositories

### Create your first repository secret

1. **In your `github-actions-fundamentals` repository**:
   - Click **Settings** tab
   - Scroll down to **Secrets and variables** in left sidebar
   - Click **Actions**

2. **Create a new repository secret**:
   - Click **New repository secret**
   - **Name**: `API_KEY`
   - **Value**: `sk_test_1234567890abcdef` (fake API key for testing)
   - Click **Add secret**

3. **Create more test secrets**:
   - **Name**: `DATABASE_PASSWORD`
   - **Value**: `super_secret_db_password_123`
   - Click **Add secret**

4. **Create a deployment token**:
   - **Name**: `DEPLOY_TOKEN`
   - **Value**: `ghp_fake_deployment_token_for_testing`
   - Click **Add secret**

### What just happened?

- ✅ Created encrypted secrets stored in GitHub
- ✅ These values are now accessible in workflows
- ✅ Once saved, you cannot view the value again (only update/delete)
- ✅ Secrets are redacted in logs (shown as `***`)

---

## Exercise 2: Using Secrets in Workflows (10 minutes)

### Access secrets securely in your workflows

1. **Create `.github/workflows/use-secrets.yml`**:

```yaml
name: Using Secrets Safely

on:
  workflow_dispatch:

jobs:
  demonstrate-secrets:
    runs-on: ubuntu-latest
    
    steps:
      - name: ❌ WRONG - Printing secret directly
        run: |
          echo "This is WRONG and will be redacted:"
          echo "API Key: ${{ secrets.API_KEY }}"
          echo "Password: ${{ secrets.DATABASE_PASSWORD }}"
      
      - name: ✅ CORRECT - Using secret in commands
        run: |
          echo "Connecting to database..."
          # In real scenario: mysql -u admin -p"${{ secrets.DATABASE_PASSWORD }}" -h db.example.com
          echo "Database connection successful (password used but not shown)"
      
      - name: ✅ CORRECT - Using secret as environment variable
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          echo "Making API call with authentication..."
          # In real scenario: curl -H "Authorization: Bearer $API_KEY" https://api.example.com
          echo "API call successful"
          
          echo "Deploying with token..."
          # In real scenario: deploy-cli --token "$DEPLOY_TOKEN" deploy
          echo "Deployment initiated"
      
      - name: ✅ CORRECT - Passing secrets to actions
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}  # Built-in secret
      
      - name: Check secret availability
        run: |
          if [ -z "${{ secrets.API_KEY }}" ]; then
            echo "❌ API_KEY secret not found"
            exit 1
          else
            echo "✅ API_KEY secret is available"
          fi
```

2. **Commit the workflow**: `Add secrets demonstration workflow`

3. **Run the workflow manually**:
   - Go to **Actions** → **Using Secrets Safely**
   - Click **Run workflow**

4. **Examine the logs**:
   - Notice secrets are shown as `***`
   - GitHub automatically redacts secret values
   - You can use secrets but never see them in logs

### Secret Best Practices

**DO:**
- ✅ Use secrets for all sensitive data
- ✅ Set secrets as environment variables
- ✅ Use the built-in `secrets.GITHUB_TOKEN` for GitHub API access
- ✅ Rotate secrets regularly
- ✅ Use different secrets for different environments

**DON'T:**
- ❌ Echo or print secrets to logs
- ❌ Hardcode sensitive values in workflow files
- ❌ Commit secrets to repository
- ❌ Share secrets between unrelated projects
- ❌ Use secrets in pull requests from forks (they're not available)

---

## Exercise 3: Create Development Environment (15 minutes)

### Set up your first deployment environment

1. **Create development environment**:
   - Go to **Settings** → **Environments**
   - Click **New environment**
   - **Name**: `development`
   - Click **Configure environment**

2. **Configure development environment**:
   - **Environment protection rules**: Leave unchecked (no approval needed)
   - **Environment secrets**: Click **Add secret**
     - **Name**: `DEPLOY_URL`
     - **Value**: `https://dev.example.com`
     - Click **Add secret**
   - **Add another secret**:
     - **Name**: `DATABASE_URL`
     - **Value**: `postgresql://dev-db.example.com:5432/myapp_dev`
     - Click **Add secret**

3. **Create a deployment workflow** - `.github/workflows/deploy-dev.yml`:

```yaml
name: Deploy to Development

on:
  push:
    branches: [develop]
  workflow_dispatch:

jobs:
  deploy-to-dev:
    runs-on: ubuntu-latest
    environment: development
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Display environment info
        run: |
          echo "======================================"
          echo "   DEPLOYING TO DEVELOPMENT"
          echo "======================================"
          echo ""
          echo "Environment: development"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Triggered by: ${{ github.actor }}"
      
      - name: Run tests
        run: |
          echo "Running unit tests..."
          echo "✓ All tests passed"
      
      - name: Build application
        run: |
          echo "Building application..."
          echo "Build ID: build-${{ github.run_number }}"
          echo "✓ Build completed"
      
      - name: Deploy to development server
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          echo "Deploying to $DEPLOY_URL"
          echo "Connecting to database..."
          echo "Running database migrations..."
          echo "Deploying application code..."
          echo ""
          echo "✓ Successfully deployed to development!"
          echo ""
          echo "Application URL: $DEPLOY_URL"
          echo "Deployment time: $(date)"
      
      - name: Run smoke tests
        run: |
          echo "Running smoke tests against development..."
          echo "✓ Health check passed"
          echo "✓ API endpoints responding"
          echo "✓ Database connectivity verified"
      
      - name: Notify team
        run: |
          echo "📢 Deployment notification sent to team"
          echo "Environment: development"
          echo "Status: Success ✓"
```

4. **Commit the workflow**: `Add development deployment workflow`

5. **Run the workflow manually** and observe the deployment simulation

### What just happened?

- ✅ Created an **environment** named "development"
- ✅ Added environment-specific secrets
- ✅ Used `environment: development` in the job
- ✅ Accessed environment secrets during deployment
- ✅ Simulated a complete deployment pipeline

**Key concept**: Environment secrets override repository secrets with the same name!

---

## Exercise 4: Create Staging Environment with Approvals (15 minutes)

### Add manual approval gates for staging deployments

1. **Create staging environment**:
   - Go to **Settings** → **Environments**
   - Click **New environment**
   - **Name**: `staging`
   - Click **Configure environment**

2. **Configure staging environment protection**:
   - ✅ **Required reviewers**: Check this box
   - Click in the search box and select yourself as a reviewer
   - **Wait timer**: Leave at 0 (or set to 5 minutes for delayed deployments)
   
3. **Add staging secrets**:
   - Click **Add secret**:
     - **Name**: `DEPLOY_URL`
     - **Value**: `https://staging.example.com`
   - Click **Add secret**:
     - **Name**: `DATABASE_URL`
     - **Value**: `postgresql://staging-db.example.com:5432/myapp_staging`

4. **Create staging deployment workflow** - `.github/workflows/deploy-staging.yml`:

```yaml
name: Deploy to Staging

on:
  workflow_dispatch:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run full test suite
        run: |
          echo "Running comprehensive tests..."
          echo "✓ Unit tests: 156 passed"
          echo "✓ Integration tests: 43 passed"
          echo "✓ E2E tests: 12 passed"
      
      - name: Security scan
        run: |
          echo "Running security scans..."
          echo "✓ No vulnerabilities found"
          echo "✓ Dependency check passed"
      
      - name: Build application
        run: |
          echo "Building production-ready application..."
          mkdir -p dist
          echo "Build ${{ github.run_number }}" > dist/version.txt
          echo "✓ Build completed successfully"
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: staging-build
          path: dist/
          retention-days: 7
  
  deploy-to-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    environment: staging  # This triggers approval requirement
    
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: staging-build
      
      - name: Display deployment info
        run: |
          echo "======================================"
          echo "   DEPLOYING TO STAGING"
          echo "======================================"
          echo ""
          echo "Environment: staging"
          echo "Build version: $(cat version.txt)"
          echo "Deploying to: ${{ secrets.DEPLOY_URL }}"
          echo "Approved by: ${{ github.actor }}"
      
      - name: Database backup
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "Creating database backup before deployment..."
          echo "Backup created: staging-backup-$(date +%Y%m%d-%H%M%S).sql"
          echo "✓ Backup stored safely"
      
      - name: Deploy to staging
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          echo "Deploying to staging environment..."
          echo "Running database migrations..."
          echo "Deploying application..."
          echo "Configuring environment..."
          echo ""
          echo "✓ Deployment completed successfully!"
          echo ""
          echo "Staging URL: $DEPLOY_URL"
      
      - name: Post-deployment verification
        run: |
          echo "Running post-deployment checks..."
          echo "✓ Application is responding"
          echo "✓ Database connections verified"
          echo "✓ All services healthy"
      
      - name: Deployment summary
        run: |
          echo "## Staging Deployment Summary 🚀" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment**: staging" >> $GITHUB_STEP_SUMMARY
          echo "- **Status**: ✅ Success" >> $GITHUB_STEP_SUMMARY
          echo "- **Build**: #${{ github.run_number }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by**: ${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Time**: $(date)" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**URL**: ${{ secrets.DEPLOY_URL }}" >> $GITHUB_STEP_SUMMARY
```

5. **Commit the workflow**: `Add staging deployment with approvals`

6. **Trigger the staging deployment**:
   - Go to **Actions** → **Deploy to Staging**
   - Click **Run workflow**
   - Select **main** branch
   - Click **Run workflow**

7. **Approve the deployment**:
   - The workflow will run the build-and-test job
   - Then it will **pause** at deploy-to-staging job
   - You'll see a yellow "Waiting" status
   - Click on the job
   - Click **Review deployments**
   - Check the **staging** checkbox
   - Optionally add a comment: "Approved for staging deployment"
   - Click **Approve and deploy**

8. **Watch the deployment complete** after approval

### What just happened?

- ✅ Created a **protected environment** with required reviewers
- ✅ Workflow paused and waited for manual approval
- ✅ Deployment only proceeded after explicit approval
- ✅ Created a realistic CI/CD pipeline: Build → Test → Approve → Deploy

**Real-world use**: Prevents accidental deployments, requires senior approval, audit trail

---

## Exercise 5: Create Production Environment (15 minutes)

### Set up production with maximum protection

1. **Create production environment**:
   - Go to **Settings** → **Environments**
   - Click **New environment**
   - **Name**: `production`
   - Click **Configure environment**

2. **Configure production protection (maximum security)**:
   - ✅ **Required reviewers**: Check and add yourself
   - ✅ **Wait timer**: Set to **5 minutes** (optional safety delay)
   - ✅ **Deployment branches**: Click dropdown
     - Select **Protected branches**
     - This means only protected branches can deploy to production

3. **Protect the main branch**:
   - Go to **Settings** → **Branches**
   - Click **Add branch protection rule**
   - **Branch name pattern**: `main`
   - ✅ **Require a pull request before merging**
   - ✅ **Require status checks to pass before merging**
   - Click **Create** (or **Save changes**)

4. **Add production secrets**:
   - Back in **Environments** → **production**
   - Add secret:
     - **Name**: `DEPLOY_URL`
     - **Value**: `https://app.example.com`
   - Add secret:
     - **Name**: `DATABASE_URL`
     - **Value**: `postgresql://prod-db.example.com:5432/myapp_production`
   - Add secret:
     - **Name**: `MONITORING_API_KEY`
     - **Value**: `mon_prod_1234567890`

5. **Create production deployment workflow** - `.github/workflows/deploy-production.yml`:

```yaml
name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy'
        required: true
        default: 'latest'

jobs:
  pre-deployment-checks:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Verify version
        run: |
          echo "Deployment version: ${{ github.event.inputs.version }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit SHA: ${{ github.sha }}"
      
      - name: Security audit
        run: |
          echo "Running security audit..."
          echo "✓ No critical vulnerabilities"
          echo "✓ All dependencies up to date"
          echo "✓ Security scan passed"
      
      - name: Run full test suite
        run: |
          echo "Running complete test suite..."
          echo "✓ Unit tests: 156/156 passed"
          echo "✓ Integration tests: 43/43 passed"
          echo "✓ E2E tests: 12/12 passed"
          echo "✓ Performance tests: passed"
          echo "✓ Load tests: passed"
      
      - name: Build production artifact
        run: |
          echo "Building production-ready application..."
          mkdir -p dist
          echo "Version: ${{ github.event.inputs.version }}" > dist/version.txt
          echo "Build: ${{ github.run_number }}" >> dist/version.txt
          echo "Commit: ${{ github.sha }}" >> dist/version.txt
          echo "✓ Production build completed"
      
      - name: Upload production build
        uses: actions/upload-artifact@v4
        with:
          name: production-build
          path: dist/
          retention-days: 30
  
  deploy-to-production:
    needs: pre-deployment-checks
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Download production build
        uses: actions/download-artifact@v4
        with:
          name: production-build
      
      - name: Pre-deployment announcement
        run: |
          echo "⚠️  ⚠️  ⚠️  PRODUCTION DEPLOYMENT  ⚠️  ⚠️  ⚠️"
          echo ""
          echo "Environment: PRODUCTION"
          echo "Version: ${{ github.event.inputs.version }}"
          cat version.txt
          echo ""
          echo "Approved by: ${{ github.actor }}"
          echo "Deployment URL: ${{ secrets.DEPLOY_URL }}"
      
      - name: Create production backup
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "Creating FULL production backup..."
          echo "Database backup: prod-backup-$(date +%Y%m%d-%H%M%S).sql"
          echo "File backup: prod-files-$(date +%Y%m%d-%H%M%S).tar.gz"
          echo "✓ Backups created and verified"
      
      - name: Enable maintenance mode
        run: |
          echo "Enabling maintenance mode..."
          echo "✓ Maintenance page activated"
          echo "✓ Users notified of brief downtime"
      
      - name: Deploy to production
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
          MONITORING_API_KEY: ${{ secrets.MONITORING_API_KEY }}
        run: |
          echo "Deploying to production servers..."
          echo ""
          echo "Step 1: Running database migrations..."
          echo "  ✓ Migrations completed successfully"
          echo ""
          echo "Step 2: Deploying application code..."
          echo "  ✓ Code deployed to all servers"
          echo ""
          echo "Step 3: Updating configuration..."
          echo "  ✓ Configuration updated"
          echo ""
          echo "Step 4: Restarting services..."
          echo "  ✓ All services restarted"
          echo ""
          echo "✅ PRODUCTION DEPLOYMENT SUCCESSFUL"
      
      - name: Disable maintenance mode
        run: |
          echo "Disabling maintenance mode..."
          echo "✓ Application is now live"
      
      - name: Post-deployment verification
        run: |
          echo "Running production health checks..."
          echo "✓ Application responding"
          echo "✓ Database connectivity verified"
          echo "✓ API endpoints healthy"
          echo "✓ Cache systems operational"
          echo "✓ CDN distribution verified"
      
      - name: Update monitoring
        env:
          MONITORING_API_KEY: ${{ secrets.MONITORING_API_KEY }}
        run: |
          echo "Updating monitoring systems..."
          echo "✓ New version registered in monitoring"
          echo "✓ Alerts configured"
          echo "✓ Dashboards updated"
      
      - name: Deployment summary
        run: |
          echo "## 🚀 Production Deployment Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Deployment Details" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment**: 🔴 PRODUCTION" >> $GITHUB_STEP_SUMMARY
          echo "- **Status**: ✅ SUCCESS" >> $GITHUB_STEP_SUMMARY
          echo "- **Version**: ${{ github.event.inputs.version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Build**: #${{ github.run_number }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by**: ${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Time**: $(date)" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### URLs" >> $GITHUB_STEP_SUMMARY
          echo "- **Production**: ${{ secrets.DEPLOY_URL }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Post-Deployment" >> $GITHUB_STEP_SUMMARY
          echo "- ✅ Health checks passed" >> $GITHUB_STEP_SUMMARY
          echo "- ✅ Monitoring updated" >> $GITHUB_STEP_SUMMARY
          echo "- ✅ All systems operational" >> $GITHUB_STEP_SUMMARY
      
      - name: Notify stakeholders
        run: |
          echo "📢 Sending deployment notifications..."
          echo "  ✓ Team notified via Slack"
          echo "  ✓ Stakeholders emailed"
          echo "  ✓ Status page updated"
```

6. **Commit the workflow**: `Add production deployment workflow`

7. **Test production deployment**:
   - Go to **Actions** → **Deploy to Production**
   - Click **Run workflow**
   - **Version**: Enter `v1.0.0`
   - Click **Run workflow**
   - Wait for pre-deployment checks
   - **Approve the deployment** when prompted
   - If you set a wait timer, observe the countdown
   - Watch the complete production deployment

### What just happened?

- ✅ Created **highly protected production environment**
- ✅ Added approval requirements
- ✅ Added optional wait timer (safety delay)
- ✅ Restricted to protected branches only
- ✅ Created comprehensive deployment workflow with:
  - Security checks
  - Full test suite
  - Backup creation
  - Maintenance mode
  - Health verification
  - Monitoring updates
  - Stakeholder notifications

---

## Exercise 6: Complete CI/CD Pipeline (15 minutes)

### Create a unified workflow for all environments

1. **Create a complete CI/CD workflow** - `.github/workflows/cicd-pipeline.yml`:

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        type: choice
        options:
          - development
          - staging
          - production
        default: development

jobs:
  # Stage 1: Continuous Integration
  continuous-integration:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Code quality checks
        run: |
          echo "Running linters..."
          echo "✓ ESLint passed"
          echo "✓ Prettier passed"
          echo "✓ Code style validated"
      
      - name: Security scanning
        run: |
          echo "Running security scans..."
          echo "✓ SAST scan completed"
          echo "✓ Dependency check passed"
          echo "✓ No vulnerabilities found"
      
      - name: Run tests
        run: |
          echo "Running test suite..."
          echo "✓ Unit tests: 156 passed"
          echo "✓ Integration tests: 43 passed"
          echo "✓ Coverage: 87%"
      
      - name: Build application
        run: |
          echo "Building application..."
          mkdir -p dist
          echo "Build ${{ github.run_number }}" > dist/build-info.txt
          echo "Branch: ${{ github.ref_name }}" >> dist/build-info.txt
          echo "Commit: ${{ github.sha }}" >> dist/build-info.txt
          echo "✓ Build completed"
      
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-build
          path: dist/
  
  # Stage 2: Deploy to Development (automatic)
  deploy-development:
    needs: continuous-integration
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: development
    
    steps:
      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: app-build
      
      - name: Deploy to development
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "🚀 Deploying to DEVELOPMENT"
          cat build-info.txt
          echo ""
          echo "Deploying to: $DEPLOY_URL"
          echo "✅ Development deployment complete"
  
  # Stage 3: Deploy to Staging (requires approval)
  deploy-staging:
    needs: continuous-integration
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: app-build
      
      - name: Deploy to staging
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "🚀 Deploying to STAGING"
          cat build-info.txt
          echo ""
          echo "Deploying to: $DEPLOY_URL"
          echo "✅ Staging deployment complete"
  
  # Stage 4: Deploy to Production (manual workflow_dispatch only)
  deploy-production:
    needs: continuous-integration
    if: github.event_name == 'workflow_dispatch' && github.event.inputs.environment == 'production'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: app-build
      
      - name: Deploy to production
        env:
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          MONITORING_API_KEY: ${{ secrets.MONITORING_API_KEY }}
        run: |
          echo "⚠️  DEPLOYING TO PRODUCTION ⚠️"
          cat build-info.txt
          echo ""
          echo "Deploying to: $DEPLOY_URL"
          echo "✅ Production deployment complete"
      
      - name: Deployment summary
        run: |
          echo "## 🎉 Deployment Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Stage | Status |" >> $GITHUB_STEP_SUMMARY
          echo "|-------|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| CI | ✅ Passed |" >> $GITHUB_STEP_SUMMARY
          echo "| Development | ✅ Deployed |" >> $GITHUB_STEP_SUMMARY
          echo "| Staging | ✅ Deployed |" >> $GITHUB_STEP_SUMMARY
          echo "| Production | ✅ Deployed |" >> $GITHUB_STEP_SUMMARY
```

2. **Commit the workflow**: `Add complete CI/CD pipeline`

3. **Test the pipeline**:
   - **Test CI only**: Create a pull request (CI runs, no deployment)
   - **Test development**: Push to develop branch (CI + dev deployment)
   - **Test staging**: Push to main branch (CI + staging deployment with approval)
   - **Test production**: Manual workflow_dispatch with production selected

### What just happened?

You've created a **complete CI/CD pipeline** with:
- ✅ Automated CI on all branches (PRs)
- ✅ Auto-deploy to development on develop branch
- ✅ Approval-required deploy to staging on main branch
- ✅ Manual-only deploy to production
- ✅ Different secrets per environment
- ✅ Proper security gates

---

## Exercise 7: Using the Built-in GITHUB_TOKEN (10 minutes)

### Understand and use automatic authentication

1. **Create a workflow using GITHUB_TOKEN** - `.github/workflows/use-github-token.yml`:

```yaml
name: Using GITHUB_TOKEN

on:
  workflow_dispatch:

jobs:
  use-github-token:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout with token
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Get repository information using GitHub API
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "Fetching repository info using GitHub API..."
          
          # Get repository details
          curl -s -H "Authorization: token $GITHUB_TOKEN" \
               -H "Accept: application/vnd.github.v3+json" \
               "https://api.github.com/repos/${{ github.repository }}" | \
               jq '{name: .name, stars: .stargazers_count, forks: .forks_count, open_issues: .open_issues_count}'
      
      - name: List workflow runs
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "Recent workflow runs:"
          curl -s -H "Authorization: token $GITHUB_TOKEN" \
               -H "Accept: application/vnd.github.v3+json" \
               "https://api.github.com/repos/${{ github.repository }}/actions/runs?per_page=5" | \
               jq '.workflow_runs[] | {name: .name, status: .status, conclusion: .conclusion}'
      
      - name: What GITHUB_TOKEN can do
        run: |
          echo "GITHUB_TOKEN automatically provides:"
          echo "✓ Read repository contents"
          echo "✓ Read and write checks"
          echo "✓ Read and write issues"
          echo "✓ Read and write pull requests"
          echo "✓ Read and write repository projects"
          echo ""
          echo "GITHUB_TOKEN expires when the job completes"
          echo "No need to create or manage this token manually!"
```

2. **Commit and run**: `Add GITHUB_TOKEN demonstration`

### GITHUB_TOKEN vs Personal Access Tokens

| Feature | GITHUB_TOKEN | Personal Access Token |
|---------|--------------|----------------------|
| **Creation** | Automatic | Manual |
| **Scope** | Repository only | Org-wide or user-wide |
| **Expiration** | End of job | Set by user |
| **Permissions** | Limited | Customizable |
| **Security** | Very secure | Requires careful handling |
| **Use case** | Most workflows | Cross-repo operations |

**When to use GITHUB_TOKEN**: 99% of the time!  
**When to use PAT**: Cross-repository operations, creating releases, triggering other workflows

---

## What You've Learned

Through these exercises, you've mastered secrets and environments:

### 1. **Repository Secrets** (Exercise 1)
- Created encrypted secrets in GitHub
- Secrets are never visible after creation
- Used for API keys, passwords, tokens

### 2. **Using Secrets Safely** (Exercise 2)
- Accessed secrets in workflows
- Secrets automatically redacted in logs
- Set secrets as environment variables
- Never print secrets directly

### 3. **Development Environment** (Exercise 3)
- Created deployment environments
- Environment-specific secrets
- Simulated development deployment

### 4. **Staging with Approvals** (Exercise 4)
- Required reviewers for deployments
- Manual approval gates
- Approval workflow and audit trail

### 5. **Production Protection** (Exercise 5)
- Maximum security for production
- Protected branches requirement
- Wait timers for safety
- Comprehensive deployment process

### 6. **Complete CI/CD** (Exercise 6)
- Full pipeline: CI → Dev → Staging → Production
- Conditional deployments per branch
- Different triggers per environment
- Realistic deployment workflow

### 7. **GITHUB_TOKEN** (Exercise 7)
- Built-in automatic authentication
- GitHub API access
- No manual token management

---

## Best Practices

**DO:**
- ✅ Use secrets for all sensitive data
- ✅ Create environment-specific secrets
- ✅ Require approvals for production
- ✅ Use protected branches
- ✅ Add wait timers for critical environments
- ✅ Use GITHUB_TOKEN when possible
- ✅ Rotate secrets regularly
- ✅ Use different secrets per environment

**DON'T:**
- ❌ Print secrets to logs
- ❌ Hardcode sensitive values
- ❌ Share production secrets
- ❌ Use same secrets across environments
- ❌ Deploy to production without approvals
- ❌ Skip security checks
- ❌ Use personal access tokens unnecessarily

---

## Security Checklist

Before deploying to production, ensure:

- [ ] All sensitive values stored as secrets
- [ ] Production environment has required reviewers
- [ ] Main branch is protected
- [ ] Security scans included in CI
- [ ] Tests pass before deployment
- [ ] Backups created before deployment
- [ ] Health checks after deployment
- [ ] Monitoring configured
- [ ] Rollback plan documented
- [ ] Team notified of deployments

---

## Environment Strategy Reference

### Development Environment
```yaml
environment: development
# No approvals needed
# Secrets: DEPLOY_URL, DATABASE_URL
# Auto-deploy on develop branch
# Purpose: Fast iteration and testing
```

### Staging Environment
```yaml
environment: staging
# Required reviewers: 1+
# Secrets: DEPLOY_URL, DATABASE_URL
# Auto-deploy on main branch (after approval)
# Purpose: Pre-production testing
```

### Production Environment
```yaml
environment: production
# Required reviewers: 2+
# Wait timer: 5-15 minutes
# Deployment branches: Protected only
# Secrets: DEPLOY_URL, DATABASE_URL, MONITORING_API_KEY
# Manual workflow_dispatch only
# Purpose: Live customer-facing application
```

---

## Next Steps

Now that you understand secrets and environments, continue to:
👉 [Lab 7: Working with Artifacts](./lab-07-working-artifacts.md)

---

**End of Lab 6 - Secrets and Environments**
