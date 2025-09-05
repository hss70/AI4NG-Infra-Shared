# Postman Testing Guide

## Setup Instructions

### 1. Import Collection
1. Open Postman
2. Click "Import" → "Upload Files"
3. Select `AI4NG-Auth-Collection.json`

### 2. Configure Variables
After deployment, update these collection variables:

1. **Get CloudFormation Outputs**:
   ```bash
   aws cloudformation describe-stacks --stack-name AI4NG-Auth-dev --query "Stacks[0].Outputs"
   ```

2. **Update Postman Variables**:
   - `user_pool_id` → Copy from `UserPoolId` output
   - `participant_client_id` → Copy from `ParticipantClientId` output  
   - `portal_client_id` → Copy from `PortalClientId` output

### 3. Configure AWS Authentication
For the "Admin - Create User" request, you need AWS credentials:

**Option A: Use AWS CLI Profile**
1. Install Postman AWS Auth plugin
2. Configure with your AWS profile

**Option B: Use IAM User**
1. Create IAM user with `cognito-idp:AdminCreateUser` permission
2. Add Authorization header manually:
   ```
   AWS4-HMAC-SHA256 Credential=ACCESS_KEY/DATE/REGION/cognito-idp/aws4_request, SignedHeaders=host;x-amz-date, Signature=SIGNATURE
   ```

**Option C: Use AWS Console**
Skip the "Admin - Create User" request and create users manually in AWS Console.

## Testing Flow

### Complete First Login Test
1. **Admin - Create User** (requires AWS auth)
2. **Step 1 - First Login** → Should return `NEW_PASSWORD_REQUIRED`
3. **Step 2 - Set New Password** → Should return JWT tokens
4. **Step 3 - Regular Login** → Should return JWT tokens immediately
5. **Refresh Token** → Copy refresh token from Step 2/3 response to test token refresh

### Variables Auto-Update
- The collection automatically saves the `session_token` from Step 1 for use in Step 2
- Manually copy `refresh_token` from login responses to test refresh flow
- Update `test_username`, `temp_password`, and `new_password` as needed

### Expected Responses

**Step 1 Response**:
```json
{
  "ChallengeName": "NEW_PASSWORD_REQUIRED",
  "Session": "session-token-here",
  "ChallengeParameters": {
    "USER_ID_FOR_SRP": "testuser@example.com"
  }
}
```

**Step 2 & 3 Response**:
```json
{
  "AuthenticationResult": {
    "AccessToken": "eyJhbGciOiJSUzI1NiIs...",
    "IdToken": "eyJhbGciOiJSUzI1NiIs...",
    "RefreshToken": "eyJjdHkiOiJKV1QiLCJlbmMi...",
    "TokenType": "Bearer",
    "ExpiresIn": 86400
  }
}
```

**Refresh Token Response**:
```json
{
  "AuthenticationResult": {
    "AccessToken": "eyJhbGciOiJSUzI1NiIs...",
    "IdToken": "eyJhbGciOiJSUzI1NiIs...",
    "TokenType": "Bearer",
    "ExpiresIn": 86400
  }
}
```
*Note: Refresh response doesn't include a new refresh token unless it's close to expiry*

## Troubleshooting

### Common Errors
- **NotAuthorizedException**: Check username/password
- **UserNotFoundException**: User doesn't exist, run "Admin - Create User" first
- **InvalidPasswordException**: Password doesn't meet policy requirements
- **ResourceNotFoundException**: Check User Pool ID and Client IDs

### Password Policy
- Minimum 8 characters
- Must contain uppercase letter
- Must contain lowercase letter  
- Must contain number
- Symbols optional

### Token Expiry
- **Participant Client**: 24 hours
- **Portal Client**: 8 hours
- **Refresh Token**: 30 days (participant) / 7 days (portal)