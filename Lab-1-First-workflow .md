# Lab 1: Your First Workflow

**Duration:** 45-60 minutes  
**Level:** Beginner-Intermediate   


---

## Overview

This lab takes you from zero to running workflows! You'll create your own GitHub repository, write workflow files from scratch, trigger workflows in multiple ways, and learn to debug issues. By the end, you'll have hands-on experience with the complete workflow development cycle.

All exercises are performed directly in your GitHub repository.

---

## Exercise 1: Create Your Training Repository (10 minutes)

### Set up your workspace for all fundamental labs

1. **Go to GitHub** and sign in to your account

2. **Create a new repository**:
   - Click the **+** icon (top right) → **New repository**

3. **Configure the repository**:
   - **Repository name**: `github-actions-fundamentals`
   - **Description**: `Learning GitHub Actions through hands-on labs`
   - **Visibility**: ✅ **Private** (gets unlimited free Actions minutes)
   - ✅ **Add a README file**
   - ✅ **Add .gitignore**: Select "Node" from template dropdown
   - **License**: Optional (e.g., MIT)
   - Click **Create repository**

4. **Verify your repository**:
   - You should see README.md
   - You should see .gitignore file
   - Repository is ready for workflows!

### What just happened?

You've created a dedicated repository for learning GitHub Actions. This will be your sandbox for all fundamental labs.

**Why public?**
- Unlimited free Actions minutes
- No credit card required
- Can be changed to private later

---

## Exercise 2: Your First "Hello World" Workflow (10 minutes)

### Create the simplest possible workflow

1. **In your repository, click the "Actions" tab**

2. **GitHub shows starter workflows**:
   - You'll see various templates
   - Click **"set up a workflow yourself"** or **"Skip this and set up a workflow yourself"**

3. **GitHub opens an editor** with a template. Delete all content and paste:

```yaml
name: Hello World

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  say-hello:
    runs-on: ubuntu-latest
    
    steps:
      - name: Greet the world
        run: echo "Hello, GitHub Actions!"
      
      - name: Show current date
        run: date
      
      - name: Show runner information
        run: |
          echo "Operating System: $RUNNER_OS"
          echo "Architecture: $RUNNER_ARCH"
```

4. **Commit the workflow**:
   - File name: `hello-world.yml` (already set)
   - Commit message: `Add hello world workflow`
   - Click **Commit changes**

5. **The workflow triggers automatically!** (because you pushed to main)

### What just happened?

- ✅ Created a workflow file in `.github/workflows/hello-world.yml`
- ✅ Workflow listens for two events: `push` to main and `workflow_dispatch` (manual)
- ✅ Defined one job with three steps
- ✅ The commit itself triggered the workflow!

---

## Exercise 3: View Your First Workflow Run (10 minutes)

### Navigate the Actions UI like a pro

1. **Immediately click the "Actions" tab**

2. **You should see your workflow run**:
   - Yellow dot (●) = Running
   - Green checkmark (✓) = Success
   - Red X (✗) = Failed

3. **Click on the workflow run** (commit message: "Add hello world workflow")

4. **Explore the run details page**:
   - See the commit that triggered it
   - See the event type (push)
   - See duration and timestamp
   - View the jobs diagram

5. **Click on the job name "say-hello"**

6. **Expand each step to see output**:
   - Click **"Greet the world"** → see "Hello, GitHub Actions!"
   - Click **"Show current date"** → see timestamp
   - Click **"Show runner information"** → see OS and architecture

7. **Notice the automatic steps**:
   - "Set up job" (GitHub adds this)
   - "Complete job" (GitHub adds this)

### Understanding the UI

```
Actions Tab
  └── All Workflows (list)
       └── Workflow Run (specific execution)
            └── Job Details
                 └── Step Logs
                      └── Raw Output
```

**Key information displayed:**
- Workflow name and run number
- Triggering event and actor
- Total duration
- Job status and timing
- Complete logs for debugging

---

## Exercise 4: Manual Workflow Trigger (10 minutes)

### Use workflow_dispatch to run on demand

Notice the `workflow_dispatch:` in our trigger? This enables manual runs.

1. **Go to the "Actions" tab**

2. **In the left sidebar**, click on your workflow name: **"Hello World"**

3. **You'll see a "Run workflow" button** appear on the right

4. **Click "Run workflow"**:
   - Select branch: **main**
   - Click the green **"Run workflow"** button

5. **Watch the workflow execute**:
   - Refresh if needed
   - New run appears at the top
   - Notice the trigger event: "workflow_dispatch"

6. **Click on the run to view details**

7. **Compare to the previous run**:
   - Same output
   - Different trigger event
   - Different timestamp

### Why Manual Triggers Matter

**Use cases for workflow_dispatch:**
- Test workflows without pushing code
- Run deployments on demand
- Trigger maintenance tasks
- Debug workflow issues
- Emergency hotfix deployments

---

## Exercise 5: Using GitHub Context Variables (15 minutes)

### Access runtime information in your workflows

1. **Edit your workflow file**:
   - Go to **Code** tab
   - Navigate to `.github/workflows/hello-world.yml`
   - Click the pencil icon (✏️) to edit

2. **Replace the entire content** with this enhanced version:

```yaml
name: GitHub Context Demo

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  show-context:
    runs-on: ubuntu-latest
    
    steps:
      - name: Display workflow information
        run: |
          echo "======================================="
          echo "     GITHUB ACTIONS CONTEXT INFO"
          echo "======================================="
          echo ""
          echo "Workflow: ${{ github.workflow }}"
          echo "Run Number: ${{ github.run_number }}"
          echo "Run ID: ${{ github.run_id }}"
          echo "Triggered by: ${{ github.event_name }}"
          echo "Actor: ${{ github.actor }}"
          echo ""
          echo "======================================="
          echo "     REPOSITORY INFORMATION"
          echo "======================================="
          echo ""
          echo "Repository: ${{ github.repository }}"
          echo "Owner: ${{ github.repository_owner }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Full Ref: ${{ github.ref }}"
          echo "Commit SHA: ${{ github.sha }}"
          echo "Commit message: ${{ github.event.head_commit.message }}"
      
      - name: Display runner information
        run: |
          echo "======================================="
          echo "     RUNNER INFORMATION"
          echo "======================================="
          echo ""
          echo "Operating System: ${{ runner.os }}"
          echo "Architecture: ${{ runner.arch }}"
          echo "Temp directory: ${{ runner.temp }}"
          echo "Workspace: ${{ runner.workspace }}"
      
      - name: Display job information
        run: |
          echo "======================================="
          echo "     JOB INFORMATION"
          echo "======================================="
          echo ""
          echo "Job ID: ${{ github.job }}"
          echo "Server URL: ${{ github.server_url }}"
          echo "API URL: ${{ github.api_url }}"
```

3. **Commit the changes**:
   - Message: `Add GitHub context variables`
   - Click **Commit changes**

4. **Go to Actions tab and view the run**

5. **Expand all three steps to see the rich context information**

### What just happened?

GitHub provides **context objects** accessible via `${{ context.property }}`:

| Context | Description | Example Properties |
|---------|-------------|-------------------|
| `github` | Workflow and repository info | workflow, actor, repository, ref, sha |
| `runner` | Runner environment | os, arch, temp, workspace |
| `job` | Current job | status, container |
| `steps` | Previous steps | outputs, outcome, conclusion |
| `env` | Environment variables | Custom variables |
| `secrets` | Encrypted secrets | API keys, tokens |

**Syntax**: `${{ context.property }}`

---

## Exercise 6: Checkout and Work with Repository Files (10 minutes)

### Access your repository code in workflows

1. **Create a simple file in your repository**:
   - Go to **Code** tab
   - Click **Add file** → **Create new file**
   - File name: `greet.sh`
   - Content:
   ```bash
   #!/bin/bash
   echo "Hello from a script file!"
   echo "Current directory: $(pwd)"
   echo "Files in directory:"
   ls -la
   ```
   - Commit message: `Add greeting script`
   - Click **Commit changes**

2. **Create a new workflow** that uses this file:
   - Go to **Actions** tab
   - Click **New workflow**
   - Click **set up a workflow yourself**
   - Name: `use-repo-files.yml`
   - Content:

```yaml
name: Using Repository Files

on:
  workflow_dispatch:
  push:
    branches: [ main ]

jobs:
  without-checkout:
    name: Without Checkout
    runs-on: ubuntu-latest
    
    steps:
      - name: Try to list files (will be empty)
        run: |
          echo "Current directory:"
          pwd
          echo ""
          echo "Files in directory:"
          ls -la || echo "Directory is empty"
  
  with-checkout:
    name: With Checkout
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4
      
      - name: List repository files
        run: |
          echo "Current directory:"
          pwd
          echo ""
          echo "Files in directory:"
          ls -la
          echo ""
          echo "README content:"
          cat README.md
      
      - name: Run the greeting script
        run: |
          chmod +x greet.sh
          ./greet.sh
```

3. **Commit the workflow**:
   - Message: `Add workflow to demonstrate checkout`
   - Click **Commit changes**

4. **View the workflow run**

5. **Compare the two jobs**:
   - **without-checkout**: Directory is nearly empty
   - **with-checkout**: All repository files present!

### What just happened?

The **`actions/checkout@v4`** action is essential!

**Without checkout:**
- Runner starts with a fresh, empty directory
- No repository files available
- Can only use built-in commands

**With checkout:**
- Clones your repository
- Checks out the specific commit
- Makes all files available to subsequent steps

**Rule**: Always use `actions/checkout@v4` when you need repository files!

---

## Exercise 7: Adding Workflow Inputs (15 minutes)

### Make workflows interactive with inputs

1. **Create a new workflow with inputs**:
   - Go to **Actions** → **New workflow** → **set up a workflow yourself**
   - Name: `interactive-workflow.yml`
   - Content:

```yaml
name: Interactive Workflow

on:
  workflow_dispatch:
    inputs:
      name:
        description: 'Your name'
        required: true
        default: 'GitHub User'
      
      greeting:
        description: 'Greeting style'
        type: choice
        options:
          - Formal
          - Casual
          - Enthusiastic
        default: Casual
      
      include-timestamp:
        description: 'Include timestamp?'
        type: boolean
        default: true
      
      repeat-count:
        description: 'Number of times to repeat'
        type: number
        default: 1

jobs:
  greet-user:
    runs-on: ubuntu-latest
    
    steps:
      - name: Display inputs
        run: |
          echo "Received inputs:"
          echo "  Name: ${{ github.event.inputs.name }}"
          echo "  Greeting: ${{ github.event.inputs.greeting }}"
          echo "  Include timestamp: ${{ github.event.inputs.include-timestamp }}"
          echo "  Repeat count: ${{ github.event.inputs.repeat-count }}"
      
      - name: Generate formal greeting
        if: github.event.inputs.greeting == 'Formal'
        run: echo "Good day, ${{ github.event.inputs.name }}. How may I assist you today?"
      
      - name: Generate casual greeting
        if: github.event.inputs.greeting == 'Casual'
        run: echo "Hey ${{ github.event.inputs.name }}! What's up?"
      
      - name: Generate enthusiastic greeting
        if: github.event.inputs.greeting == 'Enthusiastic'
        run: echo "HELLO ${{ github.event.inputs.name }}!!! 🎉🎉🎉"
      
      - name: Repeat greeting
        run: |
          for i in $(seq 1 ${{ github.event.inputs.repeat-count }}); do
            echo "Greeting #$i: Hello ${{ github.event.inputs.name }}!"
          done
      
      - name: Show timestamp
        if: github.event.inputs.include-timestamp == 'true'
        run: echo "Current time: $(date)"
```

2. **Commit the workflow**

3. **Run it manually with different inputs**:
   - Go to **Actions** → **Interactive Workflow**
   - Click **Run workflow**
   - **Try different combinations**:
     - Name: Your actual name
     - Greeting: Try each option (Formal, Casual, Enthusiastic)
     - Include timestamp: Toggle on/off
     - Repeat count: Try 1, 3, 5

4. **View the runs** and see how outputs change based on inputs

### Input Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Text input (default) | Name, message |
| `choice` | Dropdown selection | Environment, version |
| `boolean` | Checkbox (true/false) | Enable feature |
| `number` | Numeric input | Count, timeout |
| `environment` | Environment selection | production, staging |

**Access inputs**: `${{ github.event.inputs.input-name }}`

---

## What You've Learned

Through these exercises, you've mastered workflow creation:

### 1. **Repository Setup** (Exercise 1)
- Created dedicated training repository
- Configured for unlimited Actions minutes
- Ready for all labs

### 2. **First Workflow** (Exercise 2)
- Created workflow file in `.github/workflows/`
- Defined triggers and jobs
- Used `run:` to execute commands

### 3. **Viewing Runs** (Exercise 3)
- Navigated Actions UI
- Viewed job details and logs
- Understood run metadata

### 4. **Manual Triggers** (Exercise 4)
- Used `workflow_dispatch` event
- Triggered workflows on demand
- Compared trigger events

### 5. **Context Variables** (Exercise 5)
- Accessed workflow metadata
- Used `github.*` context
- Displayed runner information

### 6. **Checkout Action** (Exercise 6)
- Understood runner environment
- Used `actions/checkout@v4`
- Accessed repository files

### 7. **Workflow Inputs** (Exercise 7)
- Created interactive workflows
- Used different input types
- Conditional step execution

---

## Best Practices

**DO:**
- Name workflows and steps descriptively
- Always include `workflow_dispatch` for testing
- Use `actions/checkout@v4` when accessing files
- Add comments for complex logic
- Test workflows with manual triggers first
- Use context variables instead of hardcoding

**DON'T:**
- Use tabs for indentation (YAML requires spaces)
- Hardcode sensitive values (use secrets)
- Create workflows without triggers
- Forget to commit workflow files to `.github/workflows/`
- Ignore failed workflow runs
- Use `@main` or `@master` for action versions (use `@v4`, `@v3`, etc.)

---

## Troubleshooting Guide

### Issue: Workflow Not Appearing

**Symptoms**: Created workflow but don't see it in Actions tab

**Solutions**:
✓ Verify file is in `.github/workflows/` directory  
✓ Check file extension is `.yml` or `.yaml`  
✓ Ensure file is committed to repository  
✓ Refresh the Actions tab  
✓ Check repository settings → Actions → Allow actions

### Issue: Syntax Error

**Symptoms**: "Invalid workflow file" error

**Solutions**:
✓ Use 2 spaces for indentation (not tabs)  
✓ Quote strings with special characters  
✓ Validate YAML syntax: https://www.yamllint.com/  
✓ Check GitHub's error message for line number  
✓ Ensure colons have space after them: `name: value`

### Issue: Workflow Not Triggering

**Symptoms**: Pushed code but workflow didn't run

**Solutions**:
✓ Verify trigger matches your action (`push` to correct branch)  
✓ Check if you're on the correct branch  
✓ Ensure workflow file has no syntax errors  
✓ Check Actions are enabled in Settings  
✓ Try manual trigger with `workflow_dispatch`

### Issue: Step Failed

**Symptoms**: Job shows red X, step failed

**Solutions**:
✓ Click on failed step to read error message  
✓ Check command exists in runner environment  
✓ Use `actions/checkout@v4` if accessing files  
✓ Verify file paths are correct  
✓ Check for typos in commands  
✓ Add `continue-on-error: true` if failure is acceptable

---

## Common Workflow Patterns Reference

### Basic CI Workflow
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm test
```

### Multi-Job Workflow
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
  
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

### Conditional Steps
```yaml
steps:
  - name: Production only
    if: github.ref == 'refs/heads/main'
    run: echo "Running on main branch"
  
  - name: Not on main
    if: github.ref != 'refs/heads/main'
    run: echo "Not on main branch"
```

---

## Next Steps

Now that you can create and run workflows, let's dive deeper into events and triggers:
👉 [Lab 3: Events and Triggers](./lab-03-events-triggers.md)

---

**End of Lab 2 - Your First Workflow**
