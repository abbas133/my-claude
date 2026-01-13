---
name: sfdx-cli
description: Expert Salesforce CLI (sf) guidance. Use when deploying, retrieving, testing, or managing Salesforce orgs. Provides commands for deployments, scratch orgs, test execution, data operations, and CI/CD automation.
---

# Salesforce CLI (SFDX)

## Project Context

- **API Version**: 65.0
- **Primary Manifest**: `manifest/internalDb.xml`
- **Full Manifest**: `manifest/package.xml`

## Authentication

```bash
# Web login (interactive)
sf org login web --set-default --alias prod

# Sandbox login
sf org login web --alias dev-sandbox --instance-url https://test.salesforce.com

# JWT (CI/CD)
sf org login jwt \
  --client-id <CONSUMER_KEY> \
  --jwt-key-file server.key \
  --username admin@example.com \
  --alias prod-ci

# Auth URL (automation)
sf org login sfdx-url --sfdx-url-file authUrl.txt --alias restored-org
```

## Deployment Commands

```bash
# Deploy using manifest (recommended)
sf project deploy start --manifest manifest/internalDb.xml --target-org prod

# Deploy source directory
sf project deploy start --source-dir force-app --target-org prod

# Deploy with tests
sf project deploy start \
  --source-dir force-app \
  --target-org prod \
  --test-level RunLocalTests

# Validate only (check-only)
sf project deploy validate \
  --source-dir force-app \
  --target-org prod \
  --test-level RunLocalTests

# Quick deploy after validation
sf project deploy quick --job-id <JOB_ID> --target-org prod

# Preview deployment
sf project deploy preview --source-dir force-app --target-org prod
```

## Test Levels

| Level | Description | Use Case |
|-------|-------------|----------|
| NoTestRun | Skip tests | Sandbox |
| RunSpecifiedTests | Named tests | Quick validation |
| RunLocalTests | All non-managed | Production |
| RunAllTestsInOrg | All tests | Full validation |

```bash
# Run specific tests
sf project deploy start \
  --source-dir force-app \
  --test-level RunSpecifiedTests \
  --tests DB_InternalWholeSaler_Ctrlr_2024_Test \
  --tests DB_InternalWholeSaler_Utill_Test \
  --target-org prod
```

## Retrieve Commands

```bash
# Retrieve using manifest
sf project retrieve start --manifest manifest/internalDb.xml --target-org prod

# Retrieve specific metadata
sf project retrieve start --metadata ApexClass:DB_InternalWholeSaler_Controller_2024 --target-org prod

# Preview retrieval
sf project retrieve preview --manifest manifest/internalDb.xml --target-org prod
```

## Apex Commands

```bash
# Run test class
sf apex run test \
  --class-names DB_InternalWholeSaler_Ctrlr_2024_Test \
  --target-org dev \
  --code-coverage \
  --result-format human

# Run all project tests
sf apex run test \
  --class-names DB_InternalWholeSaler_Ctrlr_2024_Test,DB_InternalWholeSaler_Utill_Test \
  --target-org dev \
  --code-coverage

# Execute anonymous Apex
sf apex run --file scripts/apex/hello.apex --target-org dev

# View debug logs
sf apex tail log --target-org dev
```

## Scratch Org Management

```bash
# Create scratch org
sf org create scratch \
  --definition-file config/project-scratch-def.json \
  --alias scratch-dev \
  --duration-days 7 \
  --set-default

# List orgs
sf org list --all

# Open org
sf org open --target-org scratch-dev

# Delete scratch org
sf org delete scratch --target-org scratch-dev --no-prompt

# Generate password
sf org generate password --target-org scratch-dev
```

## Data Operations

```bash
# Query data
sf data query \
  --query "SELECT Id, Name FROM User WHERE IsActive = true LIMIT 10" \
  --target-org prod \
  --result-format json

# Export data tree
sf data export tree \
  --query "SELECT Id, Name FROM Account LIMIT 10" \
  --output-dir ./data \
  --target-org prod

# Import data
sf data import tree --files data/Account.json --target-org scratch-dev
```

## CI/CD Script

```bash
#!/bin/bash
set -e

ORG_ALIAS=$1
TEST_LEVEL=${2:-RunLocalTests}

echo "Validating deployment to $ORG_ALIAS..."
sf project deploy validate \
  --source-dir force-app \
  --test-level $TEST_LEVEL \
  --target-org $ORG_ALIAS \
  --wait 60

JOB_ID=$(sf project deploy report --target-org $ORG_ALIAS --json | jq -r '.result.id')

echo "Quick deploying..."
sf project deploy quick --job-id $JOB_ID --target-org $ORG_ALIAS

echo "Deployment complete!"
```

## Troubleshooting

```bash
# Check org limits
sf limits api display --target-org prod

# View deployment status
sf project deploy report --target-org prod

# Get deployment errors
sf project deploy report --target-org prod --json | jq '.result.details.componentFailures'

# Clear cache
sf cache clear

# Update CLI
sf update
```

## Useful Aliases

```bash
alias sfo='sf org open'
alias sfp='sf project deploy start --source-dir force-app'
alias sfr='sf project retrieve start --manifest manifest/internalDb.xml'
alias sft='sf apex run test --code-coverage'
```
