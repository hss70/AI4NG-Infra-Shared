# AI4NG Authentication Process Flow

## User Registration & First Login

### Admin Creates New User
1. **Admin** uses AWS Console or API to create new user
2. **Cognito** generates temporary password
3. **Cognito** sends invitation email to user (if enabled)
4. **User** receives email with temporary credentials

### First Login Process
1. **User** attempts login with temporary password
2. **Cognito** returns `NEW_PASSWORD_REQUIRED` challenge
3. **App** prompts user to set permanent password
4. **User** sets new password
5. **Cognito** returns JWT tokens
6. **User** is now authenticated

### Subsequent Logins
1. **User** logs in with permanent password
2. **Cognito** returns JWT tokens immediately
3. **User** accesses protected resources

## API Flow Details

### Step 1: InitiateAuth (First Login)
```
POST /
X-Amz-Target: AWSCognitoIdentityProviderService.InitiateAuth

Request:
{
  "ClientId": "participant-client-id",
  "AuthFlow": "USER_PASSWORD_AUTH",
  "AuthParameters": {
    "USERNAME": "user@example.com",
    "PASSWORD": "temporary-password"
  }
}

Response:
{
  "ChallengeName": "NEW_PASSWORD_REQUIRED",
  "Session": "session-token",
  "ChallengeParameters": {
    "USER_ID_FOR_SRP": "user@example.com"
  }
}
```

### Step 2: RespondToAuthChallenge (Set New Password)
```
POST /
X-Amz-Target: AWSCognitoIdentityProviderService.RespondToAuthChallenge

Request:
{
  "ClientId": "participant-client-id",
  "ChallengeName": "NEW_PASSWORD_REQUIRED",
  "Session": "session-token-from-step-1",
  "ChallengeResponses": {
    "USERNAME": "user@example.com",
    "NEW_PASSWORD": "new-permanent-password"
  }
}

Response:
{
  "AuthenticationResult": {
    "AccessToken": "jwt-access-token",
    "IdToken": "jwt-id-token",
    "RefreshToken": "refresh-token",
    "TokenType": "Bearer",
    "ExpiresIn": 28800
  }
}
```

### Step 3: Regular Login (After Password Set)
```
POST /
X-Amz-Target: AWSCognitoIdentityProviderService.InitiateAuth

Request:
{
  "ClientId": "participant-client-id",
  "AuthFlow": "USER_PASSWORD_AUTH",
  "AuthParameters": {
    "USERNAME": "user@example.com",
    "PASSWORD": "permanent-password"
  }
}

Response:
{
  "AuthenticationResult": {
    "AccessToken": "jwt-access-token",
    "IdToken": "jwt-id-token",
    "RefreshToken": "refresh-token",
    "TokenType": "Bearer",
    "ExpiresIn": 28800
  }
}
```

### Step 4: Refresh Token (When Access Token Expires)
```
POST /
X-Amz-Target: AWSCognitoIdentityProviderService.InitiateAuth

Request:
{
  "ClientId": "participant-client-id",
  "AuthFlow": "REFRESH_TOKEN_AUTH",
  "AuthParameters": {
    "REFRESH_TOKEN": "refresh-token-from-login"
  }
}

Response:
{
  "AuthenticationResult": {
    "AccessToken": "new-jwt-access-token",
    "IdToken": "new-jwt-id-token",
    "TokenType": "Bearer",
    "ExpiresIn": 28800
  }
}
```

## Error Handling

### Common Challenges
- `NEW_PASSWORD_REQUIRED` - User must set permanent password
- `PASSWORD_VERIFIER` - SRP authentication (not used in our flow)
- `MFA_REQUIRED` - Multi-factor authentication required

### Common Errors
- `NotAuthorizedException` - Invalid credentials
- `UserNotFoundException` - User doesn't exist
- `InvalidPasswordException` - Password doesn't meet policy
- `TooManyRequestsException` - Rate limiting

## Security Notes

- Temporary passwords expire in 7 days
- JWT tokens expire in 24 hours (participant) / 8 hours (portal)
- Refresh tokens expire in 30 days (participant) / 7 days (portal)
- All API calls must use HTTPS
- Store tokens securely on client side