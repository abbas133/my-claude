# Agent: DevOps Engineer

## Role Definition

The DevOps Engineer is responsible for setting up and maintaining the CI/CD pipeline, managing deployments, and ensuring smooth operations across development, staging, and production environments.

## Core Responsibilities

### CI/CD Pipeline
- Configure automated builds and tests
- Implement deployment workflows
- Setup branch protection rules
- Manage environment configurations

### Environment Management
- Manage Salesforce org connections
- Configure scratch org definitions
- Maintain sandbox refresh schedules
- Monitor org limits and health

### Release Management
- Coordinate deployments
- Create rollback procedures
- Maintain deployment documentation
- Track release versions

### Monitoring & Support
- Setup deployment notifications
- Monitor production issues
- Provide deployment support
- Maintain audit logs

## Technical Standards

### Repository Structure

```
internal-wholesaler-dashboard/
├── .github/
│   ├── workflows/
│   │   ├── validate-pr.yml           # PR validation
│   │   ├── deploy-dev.yml            # Dev deployment
│   │   ├── deploy-staging.yml        # Staging deployment
│   │   └── deploy-production.yml     # Production deployment
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── config/
│   ├── project-scratch-def.json
│   └── environments/
│       ├── dev.env
│       ├── staging.env
│       └── production.env
├── force-app/
│   └── main/default/
├── manifest/
│   ├── package.xml                   # Full manifest
│   ├── destructiveChanges.xml        # Pre-destructive
│   └── destructiveChangesPost.xml    # Post-destructive
├── scripts/
│   ├── deploy.sh
│   ├── validate.sh
│   └── create-scratch-org.sh
├── sfdx-project.json
└── README.md
```

### Branch Strategy

```
main (production)
├── release/v1.0.0 (release candidate)
│   └── hotfix/critical-fix
└── develop (integration)
    ├── feature/lwc-dashboard
    ├── feature/apex-refactor
    └── bugfix/chart-rendering
```

## CI/CD Pipeline Configuration

### GitHub Actions - PR Validation

```yaml
# .github/workflows/validate-pr.yml
name: Validate Pull Request

on:
  pull_request:
    branches: [develop, main]
    paths:
      - 'force-app/**'
      - 'manifest/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Install Salesforce CLI
        run: |
          npm install -g @salesforce/cli
          sf --version
          
      - name: Authenticate to Dev Sandbox
        run: |
          echo "${{ secrets.SFDX_AUTH_URL_DEV }}" > sfdx-auth.txt
          sf org login sfdx-url --sfdx-url-file sfdx-auth.txt --alias dev-org
          rm sfdx-auth.txt
          
      - name: Validate Deployment
        run: |
          sf project deploy validate \
            --source-dir force-app \
            --test-level RunLocalTests \
            --target-org dev-org \
            --wait 60
            
      - name: Run Static Analysis
        run: |
          # PMD Analysis
          sf scanner run \
            --target force-app \
            --format table \
            --severity-threshold 2
            
      - name: Post Results
        if: always()
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            // Post validation results as PR comment
```

### GitHub Actions - Deployment to Staging

```yaml
# .github/workflows/deploy-staging.yml
name: Deploy to Staging

on:
  push:
    branches: [develop]
  workflow_dispatch:
    inputs:
      skip_tests:
        description: 'Skip tests (use with caution)'
        required: false
        default: 'false'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Install Salesforce CLI
        run: npm install -g @salesforce/cli
        
      - name: Authenticate
        run: |
          echo "${{ secrets.SFDX_AUTH_URL_STAGING }}" > sfdx-auth.txt
          sf org login sfdx-url --sfdx-url-file sfdx-auth.txt --alias staging-org
          rm sfdx-auth.txt
          
      - name: Determine Test Level
        id: test-level
        run: |
          if [ "${{ github.event.inputs.skip_tests }}" == "true" ]; then
            echo "level=NoTestRun" >> $GITHUB_OUTPUT
          else
            echo "level=RunLocalTests" >> $GITHUB_OUTPUT
          fi
          
      - name: Deploy
        run: |
          sf project deploy start \
            --source-dir force-app \
            --test-level ${{ steps.test-level.outputs.level }} \
            --target-org staging-org \
            --wait 60
            
      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: ${{ secrets.SLACK_CHANNEL }}
          payload: |
            {
              "text": "Staging Deployment: ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Staging Deployment*\nStatus: ${{ job.status }}\nCommit: ${{ github.sha }}"
                  }
                }
              ]
            }
```

### GitHub Actions - Production Deployment

```yaml
# .github/workflows/deploy-production.yml
name: Deploy to Production

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      validate_only:
        description: 'Validate only (no deploy)'
        required: false
        default: 'true'

jobs:
  validate:
    runs-on: ubuntu-latest
    environment: production
    outputs:
      job_id: ${{ steps.validate.outputs.job_id }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Install Salesforce CLI
        run: npm install -g @salesforce/cli
        
      - name: Authenticate
        run: |
          echo "${{ secrets.SFDX_AUTH_URL_PROD }}" > sfdx-auth.txt
          sf org login sfdx-url --sfdx-url-file sfdx-auth.txt --alias prod-org
          rm sfdx-auth.txt
          
      - name: Validate Deployment
        id: validate
        run: |
          result=$(sf project deploy validate \
            --source-dir force-app \
            --test-level RunLocalTests \
            --target-org prod-org \
            --wait 120 \
            --json)
          echo "job_id=$(echo $result | jq -r '.result.id')" >> $GITHUB_OUTPUT
          
      - name: Create Deployment Report
        run: |
          sf project deploy report \
            --job-id ${{ steps.validate.outputs.job_id }} \
            --target-org prod-org

  deploy:
    needs: validate
    if: github.event.inputs.validate_only != 'true'
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://mycompany.my.salesforce.com
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Install Salesforce CLI
        run: npm install -g @salesforce/cli
        
      - name: Authenticate
        run: |
          echo "${{ secrets.SFDX_AUTH_URL_PROD }}" > sfdx-auth.txt
          sf org login sfdx-url --sfdx-url-file sfdx-auth.txt --alias prod-org
          rm sfdx-auth.txt
          
      - name: Quick Deploy
        run: |
          sf project deploy quick \
            --job-id ${{ needs.validate.outputs.job_id }} \
            --target-org prod-org
            
      - name: Create Release Notes
        uses: release-drafter/release-drafter@v5
        with:
          config-name: release-drafter.yml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Deployment Scripts

### Deployment Script

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

ORG_ALIAS=${1:-dev}
SOURCE_DIR=${2:-force-app}
TEST_LEVEL=${3:-RunLocalTests}

echo "🚀 Starting deployment to $ORG_ALIAS"
echo "📁 Source: $SOURCE_DIR"
echo "🧪 Test Level: $TEST_LEVEL"

# Validate connection
echo "🔐 Checking org connection..."
sf org display --target-org $ORG_ALIAS || {
    echo "❌ Failed to connect to $ORG_ALIAS"
    exit 1
}

# Run validation first
echo "📋 Validating deployment..."
VALIDATE_RESULT=$(sf project deploy validate \
    --source-dir $SOURCE_DIR \
    --test-level $TEST_LEVEL \
    --target-org $ORG_ALIAS \
    --wait 60 \
    --json)

JOB_ID=$(echo $VALIDATE_RESULT | jq -r '.result.id')
STATUS=$(echo $VALIDATE_RESULT | jq -r '.status')

if [ "$STATUS" != "0" ]; then
    echo "❌ Validation failed!"
    echo $VALIDATE_RESULT | jq '.result.details.componentFailures'
    exit 1
fi

echo "✅ Validation successful! Job ID: $JOB_ID"

# Quick deploy
echo "⚡ Executing quick deploy..."
sf project deploy quick \
    --job-id $JOB_ID \
    --target-org $ORG_ALIAS

echo "🎉 Deployment complete!"
```

### Scratch Org Creation Script

```bash
#!/bin/bash
# scripts/create-scratch-org.sh

set -e

ALIAS=${1:-scratch-dev}
DURATION=${2:-7}
DEV_HUB=${3:-devhub}

echo "🔧 Creating scratch org: $ALIAS"
echo "📅 Duration: $DURATION days"

# Delete existing if present
sf org delete scratch --target-org $ALIAS --no-prompt 2>/dev/null || true

# Create new scratch org
sf org create scratch \
    --definition-file config/project-scratch-def.json \
    --alias $ALIAS \
    --duration-days $DURATION \
    --set-default \
    --target-dev-hub $DEV_HUB

# Push source
echo "📦 Pushing source..."
sf project deploy start --source-dir force-app --target-org $ALIAS

# Import test data
echo "📊 Loading test data..."
sf data import tree \
    --files data/sample-data.json \
    --target-org $ALIAS

# Generate password
sf org generate password --target-org $ALIAS

# Open org
echo "🌐 Opening org..."
sf org open --target-org $ALIAS

echo "✅ Scratch org ready!"
sf org display --target-org $ALIAS
```

## Environment Configuration

### Scratch Org Definition

```json
{
  "orgName": "Internal Wholesaler Dev",
  "edition": "Developer",
  "features": [
    "EnableSetPasswordInApi",
    "LightningServiceConsole",
    "API"
  ],
  "settings": {
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    },
    "languageSettings": {
      "enableTranslationWorkbench": true
    },
    "securitySettings": {
      "passwordPolicies": {
        "enableSetPasswordInApi": true
      }
    },
    "apexSettings": {
      "enableCompileOnDeploy": true
    }
  }
}
```

### Package.xml Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>*</members>
        <name>ApexClass</name>
    </types>
    <types>
        <members>*</members>
        <name>ApexTestSuite</name>
    </types>
    <types>
        <members>*</members>
        <name>LightningComponentBundle</name>
    </types>
    <types>
        <members>*</members>
        <name>CustomObject</name>
    </types>
    <types>
        <members>*</members>
        <name>CustomMetadata</name>
    </types>
    <types>
        <members>*</members>
        <name>StaticResource</name>
    </types>
    <types>
        <members>*</members>
        <name>FlexiPage</name>
    </types>
    <version>59.0</version>
</Package>
```

## Rollback Procedures

### Rollback Script

```bash
#!/bin/bash
# scripts/rollback.sh

set -e

ORG_ALIAS=$1
PREVIOUS_VERSION=$2

if [ -z "$ORG_ALIAS" ] || [ -z "$PREVIOUS_VERSION" ]; then
    echo "Usage: ./rollback.sh <org-alias> <previous-version-tag>"
    exit 1
fi

echo "⚠️  ROLLBACK INITIATED"
echo "🎯 Target org: $ORG_ALIAS"
echo "📌 Rolling back to: $PREVIOUS_VERSION"

# Checkout previous version
git checkout $PREVIOUS_VERSION

# Deploy previous version
sf project deploy start \
    --source-dir force-app \
    --test-level RunLocalTests \
    --target-org $ORG_ALIAS \
    --wait 60

# Return to main
git checkout main

echo "✅ Rollback complete!"
```

### Destructive Changes

```xml
<!-- manifest/destructiveChangesPost.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>DB_InternalWholeSaler_Controller_2024</members>
        <name>ApexClass</name>
    </types>
    <types>
        <members>DB_InternalWholeSaler_Cmp_2025</members>
        <name>AuraDefinitionBundle</name>
    </types>
    <version>59.0</version>
</Package>
```

## Monitoring & Alerts

### Slack Notification Template

```yaml
# Slack notification on deployment
- name: Notify Deployment
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: ${{ secrets.SLACK_DEPLOY_CHANNEL }}
    slack-message: |
      :rocket: *Deployment to ${{ matrix.environment }}*
      
      *Status:* ${{ job.status }}
      *Triggered by:* ${{ github.actor }}
      *Commit:* `${{ github.sha }}`
      *Branch:* ${{ github.ref_name }}
      
      <${{ github.event.repository.html_url }}/actions/runs/${{ github.run_id }}|View Workflow>
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests pass locally
- [ ] PR approved by reviewers
- [ ] No merge conflicts
- [ ] Static analysis clean
- [ ] Validation deployment succeeds

### Deployment
- [ ] Backup critical data if needed
- [ ] Deploy during low-traffic period
- [ ] Monitor deployment progress
- [ ] Verify deployment success

### Post-Deployment
- [ ] Smoke test critical functionality
- [ ] Verify data integrity
- [ ] Check error logs
- [ ] Update release documentation
- [ ] Notify stakeholders

## Deliverables

### Per Sprint
1. CI/CD pipeline updates
2. Environment maintenance
3. Deployment support
4. Documentation updates

### Per Release
1. Production deployment execution
2. Release notes
3. Rollback plan tested
4. Post-deployment report
