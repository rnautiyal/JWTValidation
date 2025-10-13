# JWTValidation
Azure Application Gateway steps to validate JWT token

## Overview
This guide provides step-by-step instructions for implementing JWT (JSON Web Token) validation using Azure Application Gateway. JWT validation ensures that only authenticated and authorized requests reach your backend applications.

## Prerequisites
- Azure subscription
- Azure Application Gateway (v2 SKU)
- Azure Key Vault (for storing certificates/secrets)
- Understanding of JWT token structure (Header, Payload, Signature)

## Steps to Implement JWT Validation

### Step 1: Understand JWT Token Structure
A JWT token consists of three parts separated by dots (`.`):
- **Header**: Contains token type and signing algorithm
- **Payload**: Contains claims (user data, permissions, expiration)
- **Signature**: Ensures token integrity

Example: `header.payload.signature`

### Step 2: Configure Azure Application Gateway
1. Navigate to Azure Portal
2. Go to your Application Gateway resource
3. Select **Rewrites** under Settings
4. Create a new rewrite rule set

### Step 3: Create HTTP Header Rewrite Rules
Configure rules to extract and validate JWT tokens:

1. **Extract Authorization Header**
   - Condition: Check if Authorization header exists
   - Action: Extract Bearer token from Authorization header

2. **Decode JWT Token**
   - Use rewrite rules to decode the JWT token
   - Extract claims from the token payload

### Step 4: Set Up Backend Pool Validation
1. Configure backend settings to forward validated tokens
2. Add custom headers with extracted claims
3. Set up health probes to ensure backend availability

### Step 5: Configure Azure Key Vault Integration
1. Store signing keys/certificates in Azure Key Vault
2. Grant Application Gateway access to Key Vault
3. Configure managed identity for Application Gateway

### Step 6: Implement Token Validation Logic
Use Azure Application Gateway's built-in capabilities:

1. **Signature Verification**
   - Verify token signature using public key from Key Vault
   - Reject invalid signatures with 401 Unauthorized

2. **Expiration Check**
   - Validate `exp` claim in token payload
   - Reject expired tokens with 401 Unauthorized

3. **Issuer Validation**
   - Verify `iss` claim matches expected issuer
   - Reject tokens from untrusted issuers

4. **Audience Validation**
   - Verify `aud` claim matches expected audience
   - Ensure token is intended for your application

### Step 7: Configure WAF Rules (Optional)
If using Application Gateway with WAF:
1. Create custom WAF rules for additional JWT validation
2. Block malformed tokens
3. Rate limit authentication attempts

### Step 8: Set Up Monitoring and Logging
1. Enable Application Gateway diagnostics
2. Configure Log Analytics workspace
3. Monitor failed authentication attempts
4. Set up alerts for suspicious activities

### Step 9: Test the Configuration
1. Send valid JWT token in Authorization header
2. Verify successful request forwarding to backend
3. Test with invalid/expired tokens
4. Verify proper error responses (401/403)

### Step 10: Implement Best Practices
1. Use short token expiration times (15-60 minutes)
2. Implement token refresh mechanism
3. Use secure signing algorithms (RS256, ES256)
4. Rotate signing keys regularly
5. Implement proper error handling
6. Use HTTPS for all communications

## Example Configuration

### Sample Request Header
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.signature
```

### Rewrite Rule Example
```json
{
  "name": "JWT-Validation-Rule",
  "ruleSequence": 100,
  "conditions": [
    {
      "variable": "http_req_Authorization",
      "pattern": "Bearer\\s+(.+)",
      "ignoreCase": true
    }
  ],
  "actionSet": {
    "requestHeaderConfigurations": [
      {
        "headerName": "X-JWT-Valid",
        "headerValue": "true"
      }
    ]
  }
}
```

## Troubleshooting

### Common Issues

**Issue: 401 Unauthorized Error**
- Check if token is properly formatted
- Verify token has not expired
- Ensure signature is valid
- Confirm issuer and audience claims

**Issue: Token Not Being Validated**
- Verify rewrite rules are configured correctly
- Check Application Gateway logs
- Ensure Key Vault permissions are set
- Verify managed identity is enabled

**Issue: Performance Degradation**
- Consider caching public keys
- Optimize rewrite rules
- Check backend health
- Review Application Gateway SKU size

## Security Considerations
1. Always use HTTPS/TLS for token transmission
2. Never log complete tokens (they contain sensitive data)
3. Implement rate limiting to prevent brute force attacks
4. Use strong signing algorithms
5. Regularly update and patch Application Gateway
6. Monitor for abnormal authentication patterns
7. Implement defense in depth with multiple validation layers

## Additional Resources
- [Azure Application Gateway Documentation](https://docs.microsoft.com/azure/application-gateway/)
- [JWT.io - JWT Token Debugger](https://jwt.io/)
- [Azure Key Vault Documentation](https://docs.microsoft.com/azure/key-vault/)
- [OAuth 2.0 and OpenID Connect Protocols](https://oauth.net/2/)

## Contributing
Feel free to contribute to this guide by submitting pull requests with improvements or additional examples.

## License
This documentation is provided as-is for educational purposes.
