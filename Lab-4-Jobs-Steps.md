# Lab 4: Jobs and Steps

**Duration:** 45-60 minutes  
**Level:** Beginner  
**Prerequisites:** Labs 1-3 completed

---

## Overview

This lab focuses on understanding jobs and steps—the building blocks of GitHub Actions workflows. You'll learn how to organize work into jobs, control job execution order, share data between jobs, and optimize step execution.

All exercises are performed directly in your GitHub repository.

---

## Exercise 1: Multiple Jobs Running in Parallel (10 minutes)

### Create independent jobs that run simultaneously

1. In your `github-actions-fundamentals` repository, create `.github/workflows/parallel-jobs.yml`:

```yaml
name: Parallel Jobs

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  job-1:
    runs-on: ubuntu-latest
    
    steps:
      - name: Job 1 - Step 1
        run: |
          echo "Job 1 starting..."
          sleep 5
          echo "Job 1 completed"
  
  job-2:
    runs-on: ubuntu-latest
    
    steps:
      - name: Job 2 - Step 1
        run: |
          echo "Job 2 starting..."
          sleep 5
          echo "Job 2 completed"
  
  job-3:
    runs-on: ubuntu-latest
    
    steps:
      - name: Job 3 - Step 1
        run: |
          echo "Job 3 starting..."
          sleep 5
          echo "Job 3 completed"
```

2. Commit the workflow.

3. Go to **Actions** tab and observe the run.

4. **Watch carefully**: All three jobs start at the same time (parallel execution).

### What just happened?

By default, jobs in a workflow run in parallel. This:
- Speeds up workflow execution
- Utilizes multiple runners simultaneously
- Is ideal for independent tasks (tests, linting, builds)

Each job runs on a separate runner with a fresh environment.

---

## Exercise 2: Sequential Jobs with Dependencies (10 minutes)

### Control job execution order using `needs`

1. Create `.github/workflows/sequential-jobs.yml`:

```yaml
name: Sequential Jobs

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  setup:
    runs-on: ubuntu-latest
    
    steps:
      - name: Setup phase
        run: |
          echo "Setting up environment..."
          echo "Installing dependencies..."
          echo "Setup complete!"
  
  build:
    needs: setup
    runs-on: ubuntu-latest
    
    steps:
      - name: Build phase
        run: |
          echo "Building application..."
          echo "Compiling code..."
          echo "Build complete!"
  
  test:
    needs: build
    runs-on: ubuntu-latest
    
    steps:
      - name: Test phase
        run: |
          echo "Running tests..."
          echo "All tests passed!"
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy phase
        run: |
          echo "Deploying to production..."
          echo "Deployment successful!"
```

2. Commit and run the workflow.

3. **Observe**: Jobs run one after another:
   - `setup` runs first
   - `build` waits for `setup`, then runs
   - `test` waits for `build`, then runs
   - `deploy` waits for `test`, then runs

### Job Dependency Patterns

```yaml
jobs:
  job-a:
    runs-on: ubuntu-latest
    steps: [...]
  
  job-b:
    needs: job-a        # Waits for job-a
    runs-on: ubuntu-latest
    steps: [...]
  
  job-c:
    needs: [job-a, job-b]  # Waits for BOTH
    runs-on: ubuntu-latest
    steps: [...]
```

---

## Exercise 3: Fan-Out and Fan-In Pattern (10 minutes)

### Parallel execution followed by aggregation

1. Create `.github/workflows/fan-out-in.yml`:

```yaml
name: Fan-Out Fan-In Pattern

on:
  workflow_dispatch:

jobs:
  prepare:
    runs-on: ubuntu-latest
    
    steps:
      - name: Preparation
        run: echo "Preparing data for parallel processing..."
  
  # Fan-out: Multiple parallel jobs
  process-1:
    needs: prepare
    runs-on: ubuntu-latest
    
    steps:
      - name: Process dataset 1
        run: |
          echo "Processing dataset 1..."
          sleep 3
          echo "Dataset 1 complete"
  
  process-2:
    needs: prepare
    runs-on: ubuntu-latest
    
    steps:
      - name: Process dataset 2
        run: |
          echo "Processing dataset 2..."
          sleep 3
          echo "Dataset 2 complete"
  
  process-3:
    needs: prepare
    runs-on: ubuntu-latest
    
    steps:
      - name: Process dataset 3
        run: |
          echo "Processing dataset 3..."
          sleep 3
          echo "Dataset 3 complete"
  
  # Fan-in: Aggregate results
  aggregate:
    needs: [process-1, process-2, process-3]
    runs-on: ubuntu-latest
    
    steps:
      - name: Combine results
        run: |
          echo "Aggregating results from all processes..."
          echo "All datasets processed successfully!"
```

2. Commit and run the workflow.

3. **Observe the execution pattern**:
   - `prepare` runs first
   - `process-1`, `process-2`, `process-3` run in parallel
   - `aggregate` waits for all three, then runs

### Visual Flow

```
prepare
   ├─→ process-1 ─┐
   ├─→ process-2 ─┼─→ aggregate
   └─→ process-3 ─┘
```

This pattern is perfect for:
- Running tests across multiple platforms
- Processing data in parallel chunks
- Building multiple artifacts simultaneously

---

## Exercise 4: Sharing Data Between Jobs (15 minutes)

### Use outputs to pass data from one job to another

1. Create `.github/workflows/job-outputs.yml`:

```yaml
name: Job Outputs

on:
  workflow_dispatch:

jobs:
  generate-data:
    runs-on: ubuntu-latest
    
    outputs:
      build-number: ${{ steps.generate.outputs.build-number }}
      version: ${{ steps.generate.outputs.version }}
      timestamp: ${{ steps.generate.outputs.timestamp }}
    
    steps:
      - name: Generate build information
        id: generate
        run: |
          BUILD_NUM=$((RANDOM % 1000))
          VERSION="1.0.$BUILD_NUM"
          TIMESTAMP=$(date +%s)
          
          echo "build-number=$BUILD_NUM" >> $GITHUB_OUTPUT
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "timestamp=$TIMESTAMP" >> $GITHUB_OUTPUT
          
          echo "Generated Build #$BUILD_NUM"
          echo "Version: $VERSION"
  
  use-data:
    needs: generate-data
    runs-on: ubuntu-latest
    
    steps:
      - name: Display build information
        run: |
          echo "Using build information from previous job:"
          echo "Build Number: ${{ needs.generate-data.outputs.build-number }}"
          echo "Version: ${{ needs.generate-data.outputs.version }}"
          echo "Timestamp: ${{ needs.generate-data.outputs.timestamp }}"
      
      - name: Create build artifact
        run: |
          echo "Creating artifact for version ${{ needs.generate-data.outputs.version }}"
          echo "Build #${{ needs.generate-data.outputs.build-number }}" > build.txt
          cat build.txt
```

2. Commit and run the workflow.

3. **Observe**: The `use-data` job accesses outputs from `generate-data`.

### How Job Outputs Work

1. **In the producing job**: Define outputs at job level
2. **In steps**: Write to `$GITHUB_OUTPUT`
3. **In consuming job**: Reference via `needs.job-name.outputs.output-name`

```yaml
jobs:
  producer:
    outputs:
      my-output: ${{ steps.step-id.outputs.my-output }}
    steps:
      - id: step-id
        run: echo "my-output=value" >> $GITHUB_OUTPUT
  
  consumer:
    needs: producer
    steps:
      - run: echo ${{ needs.producer.outputs.my-output }}
```

---

## Exercise 5: Conditional Job Execution (10 minutes)

### Use `if` conditions to control job execution

1. Create `.github/workflows/conditional-jobs.yml`:

```yaml
name: Conditional Jobs

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  validate:
    runs-on: ubuntu-latest
    
    steps:
      - name: Validate deployment
        run: |
          echo "Validating deployment to ${{ github.event.inputs.environment }}"
          echo "Validation passed!"
  
  deploy-dev:
    needs: validate
    if: github.event.inputs.environment == 'development'
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to development
        run: echo "🚀 Deploying to development environment"
  
  deploy-staging:
    needs: validate
    if: github.event.inputs.environment == 'staging'
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to staging
        run: echo "🚀 Deploying to staging environment"
  
  deploy-production:
    needs: validate
    if: github.event.inputs.environment == 'production'
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to production
        run: |
          echo "⚠️  PRODUCTION DEPLOYMENT"
          echo "🚀 Deploying to production environment"
  
  notify:
    needs: [deploy-dev, deploy-staging, deploy-production]
    if: always()
    runs-on: ubuntu-latest
    
    steps:
      - name: Send notification
        run: |
          echo "Deployment workflow completed"
          echo "Target: ${{ github.event.inputs.environment }}"
```

2. Commit the workflow.

3. **Test different inputs**:
   - Run with `development` - only `deploy-dev` executes
   - Run with `staging` - only `deploy-staging` executes
   - Run with `production` - only `deploy-production` executes

4. **Notice**: The `notify` job always runs due to `if: always()`

### Conditional Functions

| Function | When Job Runs |
|----------|---------------|
| `if: success()` | Previous jobs succeeded (default) |
| `if: failure()` | At least one previous job failed |
| `if: always()` | Runs regardless of previous job status |
| `if: cancelled()` | Workflow was cancelled |

---

## Exercise 6: Steps with Conditions (10 minutes)

### Control individual step execution

1. Create `.github/workflows/conditional-steps.yml`:

```yaml
name: Conditional Steps

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  conditional-steps:
    runs-on: ubuntu-latest
    
    steps:
      - name: Always runs
        run: echo "This step always executes"
      
      - name: Only on push
        if: github.event_name == 'push'
        run: echo "This step only runs on push events"
      
      - name: Only on manual trigger
        if: github.event_name == 'workflow_dispatch'
        run: echo "This step only runs on manual triggers"
      
      - name: Only on main branch
        if: github.ref == 'refs/heads/main'
        run: echo "This step only runs on main branch"
      
      - name: Simulate test
        id: test
        run: |
          # Randomly pass or fail
          if [ $((RANDOM % 2)) -eq 0 ]; then
            echo "result=success" >> $GITHUB_OUTPUT
            echo "Tests passed!"
          else
            echo "result=failure" >> $GITHUB_OUTPUT
            echo "Tests failed!"
            exit 1
          fi
        continue-on-error: true
      
      - name: Run only if tests passed
        if: steps.test.outputs.result == 'success'
        run: echo "✓ Proceeding with deployment"
      
      - name: Run only if tests failed
        if: steps.test.outputs.result == 'failure'
        run: echo "✗ Skipping deployment due to test failure"
      
      - name: Cleanup (always runs)
        if: always()
        run: echo "Performing cleanup..."
```

2. Commit and run the workflow multiple times.

3. **Observe**: Different steps run based on conditions.

### Step-Level Conditions

```yaml
steps:
  - name: Conditional step
    if: github.event_name == 'push'
    run: echo "Condition met"
  
  - name: Reference previous step
    if: steps.previous-step.outcome == 'success'
    run: echo "Previous step succeeded"
  
  - name: Multiple conditions
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    run: echo "On main AND push event"
```

---

## Exercise 7: Continue on Error (10 minutes)

### Handle step failures gracefully

1. Create `.github/workflows/error-handling.yml`:

```yaml
name: Error Handling

on:
  workflow_dispatch:

jobs:
  handle-errors:
    runs-on: ubuntu-latest
    
    steps:
      - name: Step 1 - Success
        run: echo "Step 1 executes successfully"
      
      - name: Step 2 - Will fail but continue
        continue-on-error: true
        run: |
          echo "This step will fail..."
          exit 1
      
      - name: Step 3 - Still runs
        run: echo "Step 3 runs even though step 2 failed"
      
      - name: Step 4 - Another failure
        id: failing-step
        continue-on-error: true
        run: exit 1
      
      - name: Step 5 - Check previous step
        if: steps.failing-step.outcome == 'failure'
        run: echo "Previous step failed as expected"
      
      - name: Step 6 - Final step
        if: always()
        run: echo "This always runs for cleanup"
```

2. Commit and run the workflow.

3. **Observe**:
   - Steps 2 and 4 fail (shown with ⚠️ warning icon)
   - Subsequent steps still execute
   - Workflow completes successfully overall

### Error Handling Options

```yaml
steps:
  - name: May fail
    continue-on-error: true  # Step can fail, workflow continues
    run: exit 1
  
  - name: Timeout
    timeout-minutes: 5       # Max time for this step
    run: long-running-command
  
  - name: Check outcome
    if: steps.previous.outcome == 'failure'
    run: echo "Previous step failed"
  
  - name: Check conclusion
    if: steps.previous.conclusion == 'success'
    run: echo "Previous step succeeded (even if continued on error)"
```

**Note**: `outcome` vs `conclusion`:
- `outcome`: Actual result (success/failure)
- `conclusion`: Final result after `continue-on-error` (may be success even if failed)

---

## What You've Learned

Through these exercises, you've mastered jobs and steps:

### 1. **Parallel Jobs** (Exercise 1)
- Jobs run in parallel by default
- Each job gets a fresh runner environment
- Ideal for independent tasks

### 2. **Sequential Jobs** (Exercise 2)
- Use `needs:` to create dependencies
- Jobs wait for dependencies to complete
- Build pipeline stages

### 3. **Fan-Out/Fan-In** (Exercise 3)
- Combine parallel and sequential execution
- Process multiple items simultaneously
- Aggregate results at the end

### 4. **Job Outputs** (Exercise 4)
- Share data between jobs using outputs
- Define outputs at job level
- Access via `needs.job-name.outputs.name`

### 5. **Conditional Jobs** (Exercise 5)
- Use `if:` to control job execution
- Choose jobs based on inputs or events
- Use `always()`, `failure()`, etc.

### 6. **Conditional Steps** (Exercise 6)
- Control individual steps with conditions
- Reference previous step results
- Combine multiple conditions

### 7. **Error Handling** (Exercise 7)
- Use `continue-on-error` for non-critical steps
- Check step outcomes and conclusions
- Ensure cleanup runs with `if: always()`

---

## Best Practices

**DO:**
- Use parallel jobs when possible for speed
- Add `needs:` only when necessary (creates dependencies)
- Use descriptive job and step names
- Add `if: always()` to cleanup steps
- Set reasonable `timeout-minutes` for long-running steps

**DON'T:**
- Create unnecessary job dependencies (slows execution)
- Overuse `continue-on-error` (may hide real issues)
- Share data via artifacts when outputs suffice
- Forget that each job is a fresh environment
- Create circular dependencies (job-a needs job-b, job-b needs job-a)

---

## Job and Step Patterns Reference

### Parallel Execution
```yaml
jobs:
  job-a:
    runs-on: ubuntu-latest
  job-b:
    runs-on: ubuntu-latest
  job-c:
    runs-on: ubuntu-latest
```

### Sequential Pipeline
```yaml
jobs:
  stage-1:
    runs-on: ubuntu-latest
  stage-2:
    needs: stage-1
  stage-3:
    needs: stage-2
```

### Multiple Dependencies
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
  test:
    runs-on: ubuntu-latest
  deploy:
    needs: [build, test]
```

---

## Next Steps

Now that you understand jobs and steps, continue to:
👉 [Lab 5: Using Actions from Marketplace](./lab-05-marketplace-actions.md)

---

**End of Lab 4 - Jobs and Steps**
