# CodePipeline Deployment Guide

## Overview
This project uses AWS CodePipeline for automated deployment of the AI4NG authentication infrastructure.

**⚠️ IMPORTANT**: This service manages user authentication. Changes to production affect all AI4NG users.

## Initial Setup

### 1. Create GitHub Token
1. Go to https://github.com/settings/tokens
2. Click "Developer settings" → "Personal access tokens" → "Tokens (classic)"
3. Click "Generate new token (classic)"
4. Configure:
   - **Note**: "CodePipeline access"
   - **Expiration**: 90 days (recommended)
   - **Scopes**: Check `repo` only
5. Copy the token (starts with `ghp_`)

### 2. Deploy Pipeline
```bash
aws cloudformation create-stack \
  --stack-name ai4ng-auth-pipeline \
  --template-body file://ci-cd/pipeline.yaml \
  --parameters ParameterKey=GitHubToken,ParameterValue=ghp_your_token_here \
  --capabilities CAPABILITY_IAM
```

### 3. Verify Deployment
- Check AWS Console → CodePipeline
- Pipeline should trigger automatically on code push

## How It Works

1. **Source Stage**: Pulls code from GitHub when you push to `main`
2. **Build Stage**: Runs `buildspec.yml` using CodeBuild
   - Installs SAM CLI
   - Builds and packages template
3. **DeployDev Stage**: Auto-deploys to dev environment (`AI4NG-Auth-dev`)
4. **ApprovalForProd Stage**: Manual approval gate with warning
5. **DeployProd Stage**: Deploys to production (`AI4NG-Auth-prod`)

## Deployment Order

Deploy authentication **FIRST** before other AI4NG services:

1. **AI4NG-Auth** (this repo)
2. AI4NG-SharedAPI
3. AI4NGClassifierLambda
4. AI4NGUploadLambda
5. AI4NG_T1_TA_TM

## Troubleshooting

### Pipeline Fails at Source Stage
**Cause**: GitHub token expired or invalid

**Fix**:
```bash
aws cloudformation update-stack \
  --stack-name ai4ng-auth-pipeline \
  --template-body file://ci-cd/pipeline.yaml \
  --parameters ParameterKey=GitHubToken,ParameterValue=ghp_new_token_here \
  --capabilities CAPABILITY_IAM
```

### Manual Approval
**To approve production deployment**:
1. Go to AWS Console → CodePipeline → ai4ng-auth-pipeline
2. Click "Review" on the ApprovalForProd stage
3. **READ THE WARNING** - this affects all users
4. Add comments and click "Approve"

### User Pool Changes
**Common Issues**:
- User pool modifications may require user re-authentication
- Check Cognito Console for user pool status
- Verify JWT issuer URLs in dependent services

## Manual Operations

### Trigger Pipeline Manually
AWS Console → CodePipeline → ai4ng-auth-pipeline → "Release change"

### View User Pools
AWS Console → Cognito → User pools → AI4NG-UserPool-dev/prod

### Delete Pipeline
```bash
aws cloudformation delete-stack --stack-name ai4ng-auth-pipeline
```

## Files
- `ci-cd/pipeline.yaml` - CodePipeline CloudFormation template
- `ci-cd/buildspec.yml` - CodeBuild instructions
- `infra/template.yaml` - Cognito infrastructure template