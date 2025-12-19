# Lab 6: Docker Actions

**Trainer:** GitHub Actions Intermediate Training for Enterprise  
**Duration:** 60-90 minutes  
**Prerequisites:** Docker basics, understanding of containerization, GitHub Actions knowledge

---

## Learning Objectives

By the end of this lab, you will be able to:
- Create custom Docker-based actions
- Write Dockerfiles optimized for GitHub Actions
- Use any programming language in Docker actions
- Handle inputs and outputs in containerized actions
- Pass environment variables to containers
- Debug and troubleshoot Docker actions

---

## Exercise 1: Security Scanner Docker Action (25 minutes)

### Scenario
Create a Docker action that scans code for common security issues like hardcoded secrets, API keys, and passwords. This is critical for financial institutions to prevent credential leaks.

### Task
Build a Docker action using shell scripts and common security tools to perform automated security checks.

### Steps

1. Click **Add file** → **Create new file**, name it `.github/actions/security-scanner/action.yml`.

2. Paste this content:

```yaml
name: 'Security Scanner'
description: 'Scan code for hardcoded secrets, API keys, and security issues'
author: 'Training Team'

inputs:
  scan-path:
    description: 'Path to scan (relative to repository root)'
    required: false
    default: '.'
  fail-on-findings:
    description: 'Fail the workflow if security issues are found'
    required: false
    default: 'true'
  severity-threshold:
    description: 'Minimum severity to report (low, medium, high, critical)'
    required: false
    default: 'medium'

outputs:
  findings-count:
    description: 'Number of security findings'
  critical-count:
    description: 'Number of critical findings'
  high-count:
    description: 'Number of high severity findings'
  scan-status:
    description: 'Overall scan status (passed/failed)'

runs:
  using: 'docker'
  image: 'Dockerfile'
```

3. Create `Dockerfile`:

```dockerfile
FROM alpine:3.18

# Install required tools
RUN apk add --no-cache \
    bash \
    grep \
    findutils \
    git \
    curl \
    jq

# Copy scanner scripts
COPY entrypoint.sh /entrypoint.sh
COPY patterns.json /patterns.json

# Make scripts executable
RUN chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]
```

4. Click **Commit changes** → **Commit changes**.

5. Create security patterns file: Click **Add file** → **Create new file**, name it `.github/actions/security-scanner/patterns.json`.

6. Paste this content:

```json
{
  "patterns": [
    {
      "name": "AWS Access Key",
      "pattern": "AKIA[0-9A-Z]{16}",
      "severity": "critical",
      "description": "Hardcoded AWS Access Key detected"
    },
    {
      "name": "Private Key",
      "pattern": "-----BEGIN (RSA|DSA|EC|OPENSSH) PRIVATE KEY-----",
      "severity": "critical",
      "description": "Private key file detected"
    },
    {
      "name": "Generic API Key",
      "pattern": "[aA][pP][iI][-_]?[kK][eE][yY]\\s*[:=]\\s*['\"][a-zA-Z0-9]{20,}['\"]",
      "severity": "high",
      "description": "Potential API key in code"
    },
    {
      "name": "Generic Secret",
      "pattern": "[sS][eE][cC][rR][eE][tT]\\s*[:=]\\s*['\"][^'\"]{8,}['\"]",
      "severity": "high",
      "description": "Potential hardcoded secret"
    },
    {
      "name": "Password in Code",
      "pattern": "[pP][aA][sS][sS][wW][oO][rR][dD]\\s*[:=]\\s*['\"][^'\"]{4,}['\"]",
      "severity": "medium",
      "description": "Hardcoded password detected"
    },
    {
      "name": "Database Connection String",
      "pattern": "(mongodb|mysql|postgresql|jdbc)://[^\\s]+:[^\\s]+@",
      "severity": "high",
      "description": "Database connection string with credentials"
    },
    {
      "name": "JWT Token",
      "pattern": "eyJ[a-zA-Z0-9_-]*\\.eyJ[a-zA-Z0-9_-]*\\.[a-zA-Z0-9_-]*",
      "severity": "high",
      "description": "JWT token detected in code"
    },
    {
      "name": "Bearer Token",
      "pattern": "[bB]earer\\s+[a-zA-Z0-9_\\-\\.]{20,}",
      "severity": "high",
      "description": "Bearer token in code"
    }
  ]
}
```

7. Click **Commit changes** → **Commit changes**.

8. Create the scanner script: Click **Add file** → **Create new file**, name it `.github/actions/security-scanner/entrypoint.sh`.

9. Paste this content:

```bash
#!/bin/bash

set -e

# Colors for output
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Get inputs
SCAN_PATH="${INPUT_SCAN_PATH:-.}"
FAIL_ON_FINDINGS="${INPUT_FAIL_ON_FINDINGS:-true}"
SEVERITY_THRESHOLD="${INPUT_SEVERITY_THRESHOLD:-medium}"

echo "${BLUE}=== Enterprise Security Scanner ===${NC}"
echo "Scan Path: $SCAN_PATH"
echo "Fail on Findings: $FAIL_ON_FINDINGS"
echo "Severity Threshold: $SEVERITY_THRESHOLD"
echo ""

# Initialize counters
TOTAL_FINDINGS=0
CRITICAL_COUNT=0
HIGH_COUNT=0
MEDIUM_COUNT=0
LOW_COUNT=0

# Change to workspace directory
cd /github/workspace/$SCAN_PATH

# Create findings array
FINDINGS_FILE="/tmp/findings.txt"
> "$FINDINGS_FILE"

echo "${BLUE}Scanning for security issues...${NC}"
echo ""

# Read patterns from JSON
PATTERNS=$(jq -c '.patterns[]' /patterns.json)

# Scan files
while IFS= read -r pattern_obj; do
    NAME=$(echo "$pattern_obj" | jq -r '.name')
    PATTERN=$(echo "$pattern_obj" | jq -r '.pattern')
    SEVERITY=$(echo "$pattern_obj" | jq -r '.severity')
    DESCRIPTION=$(echo "$pattern_obj" | jq -r '.description')
    
    # Search for pattern (exclude .git directory and binary files)
    MATCHES=$(grep -r -n -E "$PATTERN" --exclude-dir=.git --exclude-dir=node_modules \
              --exclude="*.jpg" --exclude="*.png" --exclude="*.gif" --exclude="*.pdf" \
              . 2>/dev/null || true)
    
    if [ -n "$MATCHES" ]; then
        COUNT=$(echo "$MATCHES" | wc -l | tr -d ' ')
        TOTAL_FINDINGS=$((TOTAL_FINDINGS + COUNT))
        
        case $SEVERITY in
            critical) CRITICAL_COUNT=$((CRITICAL_COUNT + COUNT)) ;;
            high) HIGH_COUNT=$((HIGH_COUNT + COUNT)) ;;
            medium) MEDIUM_COUNT=$((MEDIUM_COUNT + COUNT)) ;;
            low) LOW_COUNT=$((LOW_COUNT + COUNT)) ;;
        esac
        
        # Color based on severity
        case $SEVERITY in
            critical|high) COLOR=$RED ;;
            medium) COLOR=$YELLOW ;;
            *) COLOR=$NC ;;
        esac
        
        echo "${COLOR}[$SEVERITY] $NAME - $COUNT finding(s)${NC}"
        echo "$DESCRIPTION"
        echo "$MATCHES" | head -3
        echo ""
        
        # Save to findings file
        echo "[$SEVERITY] $NAME: $COUNT finding(s) - $DESCRIPTION" >> "$FINDINGS_FILE"
    fi
done <<< "$PATTERNS"

# Additional checks: Large files (potential data exfiltration)
echo "${BLUE}Checking for unusually large files...${NC}"
LARGE_FILES=$(find . -type f -size +10M -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null || true)
if [ -n "$LARGE_FILES" ]; then
    LARGE_COUNT=$(echo "$LARGE_FILES" | wc -l | tr -d ' ')
    echo "${YELLOW}[medium] Found $LARGE_COUNT file(s) larger than 10MB${NC}"
    echo "$LARGE_FILES" | head -3
    TOTAL_FINDINGS=$((TOTAL_FINDINGS + LARGE_COUNT))
    MEDIUM_COUNT=$((MEDIUM_COUNT + LARGE_COUNT))
    echo ""
fi

# Check for common sensitive file names
echo "${BLUE}Checking for sensitive filenames...${NC}"
SENSITIVE_FILES=$(find . -type f \( -name "*.pem" -o -name "*.key" -o -name "*_rsa" \
                  -o -name "*.pfx" -o -name "*.p12" -o -name ".env" \
                  -o -name "credentials.*" -o -name "*secret*" \) \
                  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null || true)
if [ -n "$SENSITIVE_FILES" ]; then
    SENSITIVE_COUNT=$(echo "$SENSITIVE_FILES" | wc -l | tr -d ' ')
    echo "${RED}[critical] Found $SENSITIVE_COUNT sensitive file(s)${NC}"
    echo "$SENSITIVE_FILES"
    TOTAL_FINDINGS=$((TOTAL_FINDINGS + SENSITIVE_COUNT))
    CRITICAL_COUNT=$((CRITICAL_COUNT + SENSITIVE_COUNT))
    echo ""
fi

# Generate summary
echo ""
echo "${BLUE}=== Scan Summary ===${NC}"
echo "Total Findings: $TOTAL_FINDINGS"
echo "  Critical: $CRITICAL_COUNT"
echo "  High: $HIGH_COUNT"
echo "  Medium: $MEDIUM_COUNT"
echo "  Low: $LOW_COUNT"
echo ""

# Set outputs
echo "findings-count=$TOTAL_FINDINGS" >> $GITHUB_OUTPUT
echo "critical-count=$CRITICAL_COUNT" >> $GITHUB_OUTPUT
echo "high-count=$HIGH_COUNT" >> $GITHUB_OUTPUT

# Determine scan status
SCAN_STATUS="passed"
if [ $TOTAL_FINDINGS -gt 0 ]; then
    case $SEVERITY_THRESHOLD in
        critical)
            [ $CRITICAL_COUNT -gt 0 ] && SCAN_STATUS="failed"
            ;;
        high)
            [ $((CRITICAL_COUNT + HIGH_COUNT)) -gt 0 ] && SCAN_STATUS="failed"
            ;;
        medium)
            [ $((CRITICAL_COUNT + HIGH_COUNT + MEDIUM_COUNT)) -gt 0 ] && SCAN_STATUS="failed"
            ;;
        low)
            [ $TOTAL_FINDINGS -gt 0 ] && SCAN_STATUS="failed"
            ;;
    esac
fi

echo "scan-status=$SCAN_STATUS" >> $GITHUB_OUTPUT

# Create GitHub job summary
cat << EOF >> $GITHUB_STEP_SUMMARY
## Security Scan Results

**Scan Path:** \`$SCAN_PATH\`  
**Status:** ${SCAN_STATUS^^}

### Findings by Severity

| Severity | Count |
|----------|-------|
| Critical | $CRITICAL_COUNT |
| High | $HIGH_COUNT |
| Medium | $MEDIUM_COUNT |
| Low | $LOW_COUNT |
| **Total** | **$TOTAL_FINDINGS** |

### Recommendations

EOF

if [ $TOTAL_FINDINGS -eq 0 ]; then
    echo "No security issues detected. Great job!" >> $GITHUB_STEP_SUMMARY
    echo "${GREEN}Scan passed - No security issues found!${NC}"
    exit 0
else
    cat << EOF >> $GITHUB_STEP_SUMMARY

1. Review all detected security findings above
2. Remove any hardcoded credentials immediately
3. Use GitHub Secrets for sensitive values
4. Implement secret scanning in your repository settings
5. Consider using a secrets management system (HashiCorp Vault, AWS Secrets Manager)

Refer to Enterprise security guidelines for proper credential management.
EOF
    
    if [ "$FAIL_ON_FINDINGS" = "true" ] && [ "$SCAN_STATUS" = "failed" ]; then
        echo "${RED}Scan failed - Security issues detected!${NC}"
        exit 1
    else
        echo "${YELLOW}Security issues detected but not failing build${NC}"
        exit 0
    fi
fi
```

10. Click **Commit changes** → **Commit changes**.

11. Create test workflow: Click **Add file** → **Create new file**, name it `.github/workflows/test-security-scanner.yml`.

12. Paste this content:

```yaml
name: Test Security Scanner

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Create test file with fake secret
        run: |
          mkdir -p test-scan
          cat << 'EOF' > test-scan/config.js
          // Configuration file
          const config = {
            apiKey: "test_key_1234567890abcdef",
            password: "MyPassword123",
            dbConnection: "postgresql://user:pass@localhost/db"
          };
          module.exports = config;
          EOF
      
      - name: Run security scan
        id: scan
        uses: ./.github/actions/security-scanner
        with:
          scan-path: 'test-scan'
          fail-on-findings: 'false'
          severity-threshold: 'medium'
        continue-on-error: true
      
      - name: Display scan results
        run: |
          echo "Findings: ${{ steps.scan.outputs.findings-count }}"
          echo "Critical: ${{ steps.scan.outputs.critical-count }}"
          echo "High: ${{ steps.scan.outputs.high-count }}"
          echo "Status: ${{ steps.scan.outputs.scan-status }}"
      
      - name: Clean up test file
        if: always()
        run: rm -rf test-scan
```

13. Click **Commit changes** → **Commit changes**.

14. Go to **Actions** tab → **Test Security Scanner** workflow → **Run workflow** → **Run workflow**.

15. View the workflow run to see the security scan results detecting the intentional vulnerabilities.

### What Just Happened?

You created a production-grade security scanner Docker action:
- **Pattern Matching**: Uses regex patterns to detect secrets, API keys, passwords, and sensitive data
- **Severity Levels**: Categorizes findings by severity (critical, high, medium, low)
- **Configurable Thresholds**: Can set minimum severity to fail builds
- **Comprehensive Scanning**: Checks for hardcoded credentials, large files, sensitive filenames
- **Professional Reporting**: Generates detailed GitHub job summaries with tables and recommendations
- **Docker Benefits**: Bundles multiple tools (grep, jq, find) without requiring runner installation
- **Real-world Application**: Addresses actual security concerns in financial services

This demonstrates how Docker actions excel at tasks requiring specific toolchains and complex processing.

### Expected Outcome
- Scanner detects the intentional test vulnerabilities
- Job summary displays professional security report
- Findings are categorized by severity
- Action provides actionable remediation guidance

---

## Exercise 2: Python-Based Docker Action (25 minutes)

### Scenario
Create a Docker action using Python to validate JSON files and generate reports.

### Task
Build a Docker action that uses Python for data processing.

### Steps

1. Click **Add file** → **Create new file**, name it `.github/actions/json-validator/action.yml`.

2. Paste this content:

```yaml
name: 'JSON Validator'
description: 'Validate and analyze JSON files'
author: 'Training Team'

inputs:
  json-file:
    description: 'Path to JSON file to validate'
    required: true
  schema-file:
    description: 'Path to JSON schema file (optional)'
    required: false
  strict-mode:
    description: 'Enable strict validation'
    required: false
    default: 'false'

outputs:
  valid:
    description: 'Whether the JSON is valid'
  error-count:
    description: 'Number of validation errors'
  report:
    description: 'Validation report'

runs:
  using: 'docker'
  image: 'Dockerfile'
```

3. Click **Commit changes** → **Commit changes**.

4. Create the Dockerfile: Click **Add file** → **Create new file**, name it `.github/actions/json-validator/Dockerfile`.

5. Paste this content:

```dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the validator script
COPY validator.py .

# Set the entrypoint
ENTRYPOINT ["python", "/app/validator.py"]
```

6. Click **Commit changes** → **Commit changes**.

7. Create requirements file: Click **Add file** → **Create new file**, name it `.github/actions/json-validator/requirements.txt`.

8. Paste this content:

```txt
jsonschema==4.19.1
```

9. Click **Commit changes** → **Commit changes**.

10. Create the validator script: Click **Add file** → **Create new file**, name it `.github/actions/json-validator/validator.py`.

11. Paste this content:

```python
#!/usr/bin/env python3

import json
import os
import sys
from pathlib import Path
from jsonschema import validate, ValidationError, SchemaError

def read_json_file(file_path):
    """Read and parse JSON file."""
    try:
        with open(file_path, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Error: File not found: {file_path}")
        sys.exit(1)
    except json.JSONDecodeError as e:
        print(f"Error: Invalid JSON in {file_path}: {str(e)}")
        sys.exit(1)

def validate_json(data, schema=None, strict=False):
    """Validate JSON data against schema."""
    errors = []
    
    if schema:
        try:
            validate(instance=data, schema=schema)
            print("JSON is valid against schema")
        except ValidationError as e:
            errors.append(f"Validation error: {e.message}")
            print(f"Validation error: {e.message}")
        except SchemaError as e:
            errors.append(f"Schema error: {e.message}")
            print(f"Schema error: {e.message}")
    else:
        print("INFO: No schema provided, checking JSON syntax only")
    
    return errors

def analyze_json(data):
    """Analyze JSON structure."""
    stats = {
        'type': type(data).__name__,
        'size': len(json.dumps(data)),
    }
    
    if isinstance(data, dict):
        stats['keys'] = len(data.keys())
        stats['nested_objects'] = sum(1 for v in data.values() if isinstance(v, dict))
        stats['nested_arrays'] = sum(1 for v in data.values() if isinstance(v, list))
    elif isinstance(data, list):
        stats['items'] = len(data)
        if data:
            stats['first_item_type'] = type(data[0]).__name__
    
    return stats

def set_output(name, value):
    """Set GitHub Actions output."""
    github_output = os.getenv('GITHUB_OUTPUT')
    if github_output:
        with open(github_output, 'a') as f:
            f.write(f"{name}={value}\n")

def add_to_summary(content):
    """Add to GitHub Actions job summary."""
    github_step_summary = os.getenv('GITHUB_STEP_SUMMARY')
    if github_step_summary:
        with open(github_step_summary, 'a') as f:
            f.write(content + "\n")

def main():
    # Get inputs from environment variables
    json_file = os.getenv('INPUT_JSON_FILE')
    schema_file = os.getenv('INPUT_SCHEMA_FILE')
    strict_mode = os.getenv('INPUT_STRICT_MODE', 'false').lower() == 'true'
    
    print(f"Validating JSON file: {json_file}")
    print(f"Strict mode: {strict_mode}")
    
    # Read JSON file
    data = read_json_file(json_file)
    print("JSON syntax is valid")
    
    # Read schema if provided
    schema = None
    if schema_file:
        print(f"Using schema: {schema_file}")
        schema = read_json_file(schema_file)
    
    # Validate JSON
    errors = validate_json(data, schema, strict_mode)
    
    # Analyze JSON structure
    stats = analyze_json(data)
    
    # Generate report
    is_valid = len(errors) == 0
    report = {
        'valid': is_valid,
        'error_count': len(errors),
        'errors': errors,
        'statistics': stats
    }
    
    # Set outputs
    set_output('valid', str(is_valid).lower())
    set_output('error-count', str(len(errors)))
    set_output('report', json.dumps(report))
    
    # Create job summary
    summary = f"""## JSON Validation Report

**File:** `{json_file}`  
**Status:** {'Valid' if is_valid else 'Invalid'}  
**Errors:** {len(errors)}

### Statistics
"""
    
    for key, value in stats.items():
        summary += f"- **{key.replace('_', ' ').title()}:** {value}\n"
    
    if errors:
        summary += "\n### Errors\n"
        for error in errors:
            summary += f"- {error}\n"
    
    add_to_summary(summary)
    
    # Exit with appropriate code
    sys.exit(0 if is_valid else 1)

if __name__ == '__main__':
    main()
```

12. Click **Commit changes** → **Commit changes**.

13. Create test workflow: Click **Add file** → **Create new file**, name it `.github/workflows/test-json-validator.yml`.

14. Paste this content:

```yaml
name: Test JSON Validator

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  validate:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Create test JSON
        run: |
          echo '{
            "name": "Test Project",
            "version": "1.0.0",
            "dependencies": ["action-core", "action-github"]
          }' > test.json
      
      - name: Validate JSON
        id: validate
        uses: ./.github/actions/json-validator
        with:
          json-file: 'test.json'
          strict-mode: 'false'
        continue-on-error: true
      
      - name: Display results
        run: |
          echo "Valid: ${{ steps.validate.outputs.valid }}"
          echo "Errors: ${{ steps.validate.outputs.error-count }}"
```

15. Click **Commit changes** → **Commit changes**.

16. Go to **Actions** tab → **Test JSON Validator** workflow → **Run workflow** → **Run workflow**.

17. View the workflow run to see JSON validation results.

### What Just Happened?

You built a Python-based Docker action with dependency management:
- Docker actions let you use any programming language (Python, Ruby, Go, Rust, etc.) without requiring it in the runner
- Dependencies are installed during image build using `RUN pip install -r requirements.txt` (or equivalent package manager)
- The workspace is automatically mounted at `/github/workspace`, so relative paths work correctly
- Python's rich ecosystem (jsonschema, requests, pandas, etc.) becomes available for complex processing
- `requirements.txt` pins exact versions for reproducible builds

This pattern is ideal for data processing, validation, or when you need specific Python libraries.

### Challenge
Extend the action to support multiple JSON files and generate a combined report.

---

## Exercise 3: Multi-Stage Docker Action (25 minutes)

### Scenario
Create an optimized Docker action using multi-stage builds to keep the final image small.

### Task
Build a Go-based Docker action with multi-stage build optimization.

### Steps

1. Click **Add file** → **Create new file**, name it `.github/actions/go-hello/action.yml`.

2. Paste this content:

```yaml
name: 'Go Hello World'
description: 'Optimized Docker action built with Go'
author: 'Training Team'

inputs:
  message:
    description: 'Message to display'
    required: true
  repeat:
    description: 'Number of times to repeat'
    required: false
    default: '1'

outputs:
  result:
    description: 'The generated output'

runs:
  using: 'docker'
  image: 'Dockerfile'
```

3. Click **Commit changes** → **Commit changes**.

4. Create the Dockerfile: Click **Add file** → **Create new file**, name it `.github/actions/go-hello/Dockerfile`.

5. Paste this multi-stage Dockerfile:

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy Go module files
COPY go.mod ./

# Download dependencies
RUN go mod download

# Copy source code
COPY main.go .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o action main.go

# Runtime stage
FROM alpine:3.18

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy the binary from builder
COPY --from=builder /app/action .

# Set the entrypoint
ENTRYPOINT ["./action"]
```

6. Click **Commit changes** → **Commit changes**.

7. Create Go module file: Click **Add file** → **Create new file**, name it `.github/actions/go-hello/go.mod`.

8. Paste this content:

```go
module github.com/enterprise/action

go 1.21
```

9. Click **Commit changes** → **Commit changes**.

10. Create the Go source file: Click **Add file** → **Create new file**, name it `.github/actions/go-hello/main.go`.

11. Paste this content:

```go
package main

import (
	"fmt"
	"os"
	"strconv"
	"strings"
)

func main() {
	// Get inputs from environment variables
	message := os.Getenv("INPUT_MESSAGE")
	repeatStr := os.Getenv("INPUT_REPEAT")
	
	if message == "" {
		fmt.Println("Error: message input is required")
		os.Exit(1)
	}
	
	repeat, err := strconv.Atoi(repeatStr)
	if err != nil || repeat < 1 {
		repeat = 1
	}
	
	// Generate output
	var result strings.Builder
	for i := 0; i < repeat; i++ {
		result.WriteString(fmt.Sprintf("%d. %s\n", i+1, message))
	}
	
	output := result.String()
	fmt.Print(output)
	
	// Set GitHub Actions output
	githubOutput := os.Getenv("GITHUB_OUTPUT")
	if githubOutput != "" {
		f, err := os.OpenFile(githubOutput, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
		if err == nil {
			defer f.Close()
			f.WriteString(fmt.Sprintf("result<<%s\n%s%s\n", "EOF", output, "EOF"))
		}
	}
	
	// Create job summary
	githubStepSummary := os.Getenv("GITHUB_STEP_SUMMARY")
	if githubStepSummary != "" {
		f, err := os.OpenFile(githubStepSummary, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
		if err == nil {
			defer f.Close()
			f.WriteString(fmt.Sprintf("## Action Output\n\n```\n%s```\n", output))
		}
	}
}
```

12. Click **Commit changes** → **Commit changes**.

13. Create test workflow: Click **Add file** → **Create new file**, name it `.github/workflows/test-go-action.yml`.

14. Paste this content:

```yaml
name: Test Go Docker Action

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Go action
        id: go-action
        uses: ./.github/actions/go-hello
        with:
          message: 'Hello from Enterprise Training!'
          repeat: '3'
      
      - name: Display result
        run: |
          echo "Result:"
          echo "${{ steps.go-action.outputs.result }}"
```

15. Click **Commit changes** → **Commit changes**.

16. Go to **Actions** tab → **Test Go Docker Action** workflow → **Run workflow** → **Run workflow**.

17. View the workflow run to see the optimized Docker action output.

### What Just Happened?

You optimized a Docker action using multi-stage builds:
- Multi-stage builds use separate stages: one for building (with compilers/build tools) and one for runtime (minimal)
- The final image only contains the compiled binary, not the build tools, drastically reducing size (often 10x smaller)
- Build stage uses `FROM golang:1.21-alpine AS builder` with full toolchain
- Runtime stage uses `FROM alpine:3.18` with only the compiled binary via `COPY --from=builder`
- Smaller images mean faster action startup times and reduced network transfer

Use multi-stage builds for compiled languages (Go, Rust, C++) or when build dependencies differ from runtime needs.

### Challenge
Compare the image sizes between a single-stage and multi-stage build.

---

## Exercise 4: Docker Action with Volume Mounts (20 minutes)

### Scenario
Create a Docker action that processes files in the workspace and generates reports.

### Task
Build an action that accesses workspace files through volume mounts.

### Steps

1. Click **Add file** → **Create new file**, name it `.github/actions/file-analyzer/action.yml`.

2. Paste this content:

```yaml
name: 'File Analyzer'
description: 'Analyze files in the workspace'
author: 'Training Team'

inputs:
  directory:
    description: 'Directory to analyze'
    required: false
    default: '.'
  file-extensions:
    description: 'Comma-separated file extensions to analyze'
    required: false
    default: 'md,js,py'

outputs:
  total-files:
    description: 'Total number of files analyzed'
  total-lines:
    description: 'Total lines of code'
  report-file:
    description: 'Path to detailed report'

runs:
  using: 'docker'
  image: 'Dockerfile'
```

3. Create `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /action

COPY analyze.py .

RUN chmod +x analyze.py

ENTRYPOINT ["python", "/action/analyze.py"]
```

6. Click **Commit changes** → **Commit changes**.

7. Create the analyzer script: Click **Add file** → **Create new file**, name it `.github/actions/file-analyzer/analyze.py`.

8. Paste this content:

```python
#!/usr/bin/env python3

import os
import sys
from pathlib import Path
from collections import defaultdict

def analyze_file(file_path):
    """Analyze a single file."""
    try:
        with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
            lines = f.readlines()
            return {
                'lines': len(lines),
                'size': os.path.getsize(file_path),
                'empty_lines': sum(1 for line in lines if not line.strip()),
            }
    except Exception as e:
        print(f"Warning: Could not analyze {file_path}: {str(e)}")
        return None

def analyze_directory(directory, extensions):
    """Analyze all files in directory with specified extensions."""
    stats = defaultdict(lambda: {'count': 0, 'lines': 0, 'size': 0})
    total_files = 0
    total_lines = 0
    
    for ext in extensions:
        pattern = f"**/*.{ext}"
        files = list(Path(directory).glob(pattern))
        
        for file_path in files:
            if file_path.is_file():
                result = analyze_file(file_path)
                if result:
                    stats[ext]['count'] += 1
                    stats[ext]['lines'] += result['lines']
                    stats[ext]['size'] += result['size']
                    total_files += 1
                    total_lines += result['lines']
    
    return stats, total_files, total_lines

def generate_report(stats, total_files, total_lines, output_file):
    """Generate detailed report."""
    report = "# File Analysis Report\n\n"
    report += f"**Total Files:** {total_files}\n"
    report += f"**Total Lines:** {total_lines}\n\n"
    report += "## Breakdown by Extension\n\n"
    report += "| Extension | Files | Lines | Size (bytes) |\n"
    report += "|-----------|-------|-------|-------------|\n"
    
    for ext, data in sorted(stats.items()):
        report += f"| .{ext} | {data['count']} | {data['lines']} | {data['size']:,} |\n"
    
    with open(output_file, 'w') as f:
        f.write(report)
    
    return report

def set_output(name, value):
    """Set GitHub Actions output."""
    github_output = os.getenv('GITHUB_OUTPUT')
    if github_output:
        with open(github_output, 'a') as f:
            # Handle multiline values
            if '\n' in str(value):
                f.write(f"{name}<<EOF\n{value}\nEOF\n")
            else:
                f.write(f"{name}={value}\n")

def main():
    # Get inputs
    directory = os.getenv('INPUT_DIRECTORY', '.')
    extensions_str = os.getenv('INPUT_FILE_EXTENSIONS', 'md,js,py')
    extensions = [ext.strip() for ext in extensions_str.split(',')]
    
    print(f"Analyzing directory: {directory}")
    print(f"Extensions: {', '.join(extensions)}")
    
    # Analyze files
    stats, total_files, total_lines = analyze_directory(directory, extensions)
    
    # Generate report
    report_file = 'file-analysis-report.md'
    report = generate_report(stats, total_files, total_lines, report_file)
    
    print(f"\nAnalysis complete!")
    print(f"Total files: {total_files}")
    print(f"Total lines: {total_lines}")
    
    # Set outputs
    set_output('total-files', total_files)
    set_output('total-lines', total_lines)
    set_output('report-file', report_file)
    
    # Add to job summary
    github_step_summary = os.getenv('GITHUB_STEP_SUMMARY')
    if github_step_summary:
        with open(github_step_summary, 'a') as f:
            f.write(report)

if __name__ == '__main__':
    main()
```

9. Click **Commit changes** → **Commit changes**.

10. Create test workflow: Click **Add file** → **Create new file**, name it `.github/workflows/test-analyzer.yml`.

11. Paste this content:

```yaml
name: Test File Analyzer

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Analyze workspace files
        id: analyze
        uses: ./.github/actions/file-analyzer
        with:
          directory: '.'
          file-extensions: 'md,yml,yaml,js,py'
      
      - name: Display results
        run: |
          echo "Total files: ${{ steps.analyze.outputs.total-files }}"
          echo "Total lines: ${{ steps.analyze.outputs.total-lines }}"
      
      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: analysis-report
          path: ${{ steps.analyze.outputs.report-file }}
```

12. Click **Commit changes** → **Commit changes**.

13. Go to **Actions** tab → **Test File Analyzer** workflow → **Run workflow** → **Run workflow**.

14. View the workflow run and download the analysis report artifact.

### What Just Happened?

You created a Docker action that processes workspace files:
- GitHub Actions automatically mounts the workspace at `/github/workspace` inside the container
- Relative paths in inputs work correctly because the working directory is set to the workspace mount point
- Files created by the action are visible to subsequent steps in the workflow
- Volume mounts are read-write, allowing actions to modify, create, or analyze repository files
- Path separators and permissions are handled transparently across Linux, macOS, and Windows runners

This pattern enables file analysis, code generation, report creation, and workspace transformations.

---

## Exercise 5: Debugging Docker Actions (15 minutes)

### Scenario
Learn techniques for debugging Docker actions when they fail.

### Task
Add debugging capabilities to a Docker action.

### Steps

1. Create `Dockerfile` with debugging support:

```dockerfile
FROM python:3.11-slim

# Install debugging tools
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /action

COPY debug-action.py .

# Add environment variable for debug mode
ENV DEBUG=false

ENTRYPOINT ["python", "/action/debug-action.py"]
```

2. Create `debug-action.py`:

```python
#!/usr/bin/env python3

import os
import sys
import json

def debug_print(message):
    """Print debug information if debug mode is enabled."""
    if os.getenv('DEBUG', 'false').lower() == 'true':
        print(f"DEBUG: {message}", file=sys.stderr)

def main():
    debug_print("Action started")
    
    # Print all environment variables in debug mode
    if os.getenv('DEBUG', 'false').lower() == 'true':
        debug_print("Environment variables:")
        for key, value in sorted(os.environ.items()):
            if key.startswith('INPUT_') or key.startswith('GITHUB_'):
                debug_print(f"  {key}={value}")
    
    # Get inputs
    name = os.getenv('INPUT_NAME', 'Unknown')
    debug_print(f"Processing name: {name}")
    
    # Simulate processing
    result = f"Hello {name}!"
    
    debug_print(f"Generated result: {result}")
    
    print(result)
    
    debug_print("Action completed successfully")

if __name__ == '__main__':
    main()
```

3. Create test workflow with debugging:

```yaml
name: Debug Docker Action

on:
  workflow_dispatch:

jobs:
  test-normal:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run without debug
        uses: ./.github/actions/debug-action
        with:
          name: 'Normal Mode'
  
  test-debug:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run with debug
        uses: ./.github/actions/debug-action
        with:
          name: 'Debug Mode'
        env:
          DEBUG: 'true'
```

### Debugging Tips
1. Use `docker build` locally to test image builds
2. Add verbose logging with environment flags
3. Use `docker run` to test the action locally
4. Check environment variables are properly passed
5. Validate file permissions and paths

---

## Best Practices for Docker Actions

### DO
- Use multi-stage builds for optimized images
- Pin base image versions for reproducibility
- Install only necessary dependencies
- Use Alpine or slim images when possible
- Handle errors gracefully
- Add comprehensive logging
- Test locally before committing
- Document required system dependencies

### DON'T
- Use `latest` tag for base images
- Install unnecessary packages
- Assume specific tools are available
- Hardcode file paths
- Ignore security vulnerabilities
- Create unnecessarily large images
- Forget to handle file permissions

---

## Docker Action Checklist

- [ ] Dockerfile builds successfully
- [ ] Base image is pinned to specific version
- [ ] Only necessary dependencies installed
- [ ] Entrypoint script is executable
- [ ] Inputs are properly handled
- [ ] Outputs are correctly set
- [ ] Error handling is comprehensive
- [ ] Image size is optimized
- [ ] Action works in workflows
- [ ] Documentation is complete

---

## Troubleshooting Guide

**Problem:** "Cannot find Dockerfile"  
**Solution:** Ensure Dockerfile is in the action directory and properly referenced in action.yml

**Problem:** Permission denied errors  
**Solution:** Make sure scripts have execute permissions (`chmod +x`)

**Problem:** Environment variables not available  
**Solution:** Check that inputs are properly prefixed with `INPUT_`

**Problem:** Large image size  
**Solution:** Use multi-stage builds and alpine/slim base images

---

## Key Takeaways

- Docker actions provide complete environment control
- Use any language or tool available in containers
- Multi-stage builds optimize image size
- Environment variables pass inputs to containers
- Workspace is automatically mounted
- Great for complex dependencies or specific tool versions
- Slower to start than JavaScript actions (image build/pull)

---

## Resources

- [Creating a Docker container action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Docker best practices](https://docs.docker.com/develop/dev-best-practices/)
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

---

**End of Lab 6**
