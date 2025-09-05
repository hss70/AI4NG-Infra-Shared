# AI4NG Authentication Service

Centralized authentication infrastructure for all AI4NG services using AWS Cognito.

## Overview

This service provides:
- **User Pool** - User registration, login, password management
- **Identity Pool** - AWS resource access for authenticated users
- **User Groups** - Role-based access control (admin, researcher, etc.)
- **JWT Tokens** - Secure API authentication across all AI4NG services

## Deployment

**Method**: AWS CodePipeline (see [ci-cd/CODEPIPELINE.md](ci-cd/CODEPIPELINE.md))

**Deploy Order**: Deploy this **first** before other AI4NG services that depend on authentication.

## Exports

This stack exports:
- `CognitoUserPoolId-dev/prod` - User Pool ID
- `CognitoParticipantClientId-dev/prod` - AI4NG Participant Mobile App Client ID
- `CognitoPortalClientId-dev/prod` - AI4NG Portal Web Client ID
- `CognitoPortalClientSecret-dev/prod` - AI4NG Portal Web Client Secret
- `CognitoIdentityPoolId-dev/prod` - Identity Pool ID
- `CognitoUserPoolArn-dev/prod` - User Pool ARN for authorizers

## Usage in Other Services

Import in your CloudFormation templates:
```yaml
# For AI4NG Participant Mobile App
JwtConfiguration:
  Audience:
    - !ImportValue CognitoParticipantClientId-dev
  Issuer: !ImportValue CognitoJwtIssuer-dev

# For AI4NG Portal Web App
JwtConfiguration:
  Audience:
    - !ImportValue CognitoPortalClientId-dev
  Issuer: !ImportValue CognitoJwtIssuer-dev
```

## Environments

- **Dev**: Separate user pool for development/testing
- **Prod**: Production user pool with real users

## User Management

Users can be managed through:
- **AWS Console** - Cognito User Pools
- **Frontend Apps** - Self-registration and login
- **Admin APIs** - Programmatic user management