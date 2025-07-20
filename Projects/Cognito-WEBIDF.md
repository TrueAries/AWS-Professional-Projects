# AWS Cognito: User/Identity Pools and Web Identity Federation Guide

## Project Overview

This project demonstrates how to implement AWS Cognito User Pools and Identity Pools with Web Identity Federation (WEBIDF) to enable secure authentication and authorization for web applications. The implementation shows how users can authenticate with external identity providers (like Google) and gain temporary AWS credentials to access AWS resources.

## Architecture Overview

<img width="1200" height="671" alt="image" src="https://github.com/user-attachments/assets/9d8268e7-670b-4260-be5b-7e2f946f0ed2" />


Amazon Cognito provides:
- **Authentication, Authorization, and User Management** for web/mobile apps
- **User Pools**: Sign-in functionality with JSON Web Tokens (JWT)
- **Identity Pools**: Access to temporary AWS credentials
- **Federated Identities**: Integration with Google, Facebook, Twitter, SAML2.0 & User Pool
- **Unauthenticated Identities**: Guest user access

## What You'll Learn

- Understanding the difference between Cognito User Pools and Identity Pools
- How Web Identity Federation works with external providers
- Setting up Google as an identity provider
- Creating and configuring Cognito Identity Pools
- Implementing temporary AWS credential access
- Testing authentication flows in a real web application

## Prerequisites

- AWS Account with appropriate permissions
- Google Developer Account
- Basic understanding of web development concepts
- Knowledge of AWS IAM roles and policies

## Architecture Components

### User Pools vs Identity Pools

<img width="1203" height="675" alt="image" src="https://github.com/user-attachments/assets/79f320a7-8d92-4599-98d4-653a8d4e2bcd" />


**User Pools**:
- Handle user authentication and return JWT tokens
- Support social sign-in with external providers
- Provide user directory management, profiles, sign-up & sign-in
- Include MFA and other security features
- Tokens can be used for API access via Lambda Custom Authorizers or API Gateway

<img width="1194" height="675" alt="image" src="https://github.com/user-attachments/assets/6e2f4ee7-678d-4fcd-ac0e-d8d844855d0f" />


**Identity Pools**:
- Provide access to temporary AWS credentials
- Support both authenticated and unauthenticated roles
- Work with external identity providers or Cognito User Pools
- Enable direct access to AWS services with proper IAM permissions

<img width="1203" height="683" alt="image" src="https://github.com/user-attachments/assets/0e539b14-0482-413d-892f-3d95f1933dee" />

**Combined Usage**:
- User Pools handle authentication and social sign-in
- Identity Pools provide AWS credentials for authenticated users
- Applications can standardize on User Pool tokens while accessing AWS resources

## Advanced Implementation Flow

<img width="1203" height="676" alt="image" src="https://github.com/user-attachments/assets/9566244b-45f3-4674-ab58-83e8c3ee6c67" />


The advanced demo shows a complete Web Identity Federation flow:
1. User accesses web application hosted on S3/CloudFront
2. User is directed to Google IdP for authentication
3. Google returns a token upon successful authentication
4. Cognito Identity Pool exchanges Google token for AWS credentials
5. AWS credentials are used to access private S3 resources
6. Application displays personalized content (like "Patches the cat" images)

## Step-by-Step Implementation

### STAGE 1: Provision the Environment and Review Tasks

#### Environment Setup
1. **S3 Bucket Creation**:
   - Create an S3 bucket named `appbucket-[random-string]`
   - Enable static website hosting
   - Configure CloudFront distribution for global access

2. **Review Architecture**:
   - Understand the flow from user authentication to resource access
   - Identify the role of each component in the architecture
   - Plan the security boundaries and access controls

#### Pre-Implementation Checklist
- [ ] AWS account ready with appropriate permissions
- [ ] S3 bucket created and configured for static hosting
- [ ] CloudFront distribution set up
- [ ] Basic HTML/JS application files prepared

### STAGE 2: Create Google API Project & Client ID

#### Google Developer Console Setup
1. **Create New Project**:
   - Navigate to [Google Cloud Console](https://console.cloud.google.com)
   - Create a new project or select existing one
   - Name: `Cognito-WEBIDF-Demo`

2. **Enable Google+ API**:
   - Go to APIs & Services → Library
   - Search for "Google+ API"
   - Enable the API for your project

3. **Create OAuth 2.0 Credentials**:
   - Navigate to APIs & Services → Credentials
   - Click "Create Credentials" → "OAuth 2.0 Client ID"
   - Application type: "Web application"
   - Name: `Cognito-WEBIDF-Client`

4. **Configure Authorized Domains**:
   - Add your CloudFront domain
   - Add localhost for testing
   - Save the Client ID (you'll need this for Cognito)

#### Important Notes
- Keep your Client ID secure but note it's not a secret for public web apps
- The Client Secret is not needed for this implementation
- Authorized domains must match exactly where your app will be hosted

### STAGE 3: Create Cognito Identity Pool

#### Identity Pool Configuration
1. **Create Identity Pool**:
   - Navigate to Amazon Cognito in AWS Console
   - Click "Create Identity Pool"
   - Identity pool name: `WEBIDF_Demo_Pool`
   - Enable unauthenticated identities: ✅

2. **Configure Authentication Providers**:
   - **Google+**:
     - Google+ App ID: [Your Google Client ID from Stage 2]
   - **Advanced Settings**:
     - Choose role from token: No
     - Role resolution: Use default role

3. **IAM Roles Creation**:
   - **Authenticated Role**: `Cognito_WEBIDF_Demo_Pool_Auth_Role`
     - Attach policy for S3 access to private objects
   - **Unauthenticated Role**: `Cognito_WEBIDF_Demo_Pool_Unauth_Role`
     - Minimal permissions (deny S3 private access)

#### IAM Policy for Authenticated Users
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::appbucket-*/private/*"
            ]
        }
    ]
}
```

#### IAM Policy for Unauthenticated Users
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::appbucket-*/private/*"
            ]
        }
    ]
}
```

### STAGE 4: Update App Bucket & Test Application

#### Application Configuration
1. **Update JavaScript Configuration**:
   ```javascript
   // Update these values in your app.js
   const GOOGLE_CLIENT_ID = 'your-google-client-id';
   const COGNITO_IDENTITY_POOL_ID = 'your-identity-pool-id';
   const AWS_REGION = 'your-aws-region';
   const S3_BUCKET = 'your-app-bucket-name';
   ```

2. **Deploy Application Files**:
   - Upload HTML, CSS, and JavaScript files to S3 bucket
   - Ensure files are publicly readable
   - Test CloudFront distribution access

#### Testing the Authentication Flow
1. **Anonymous Access Test**:
   - Access the application without signing in
   - Verify that private S3 objects are not accessible
   - Should see "Access Denied" for private content

2. **Google Authentication Test**:
   - Click "Sign in with Google"
   - Complete Google authentication flow
   - Verify redirect back to application

3. **Authenticated Access Test**:
   - After successful Google sign-in
   - Application should display private S3 content
   - Verify AWS credentials are working

#### Verification Points
- [ ] Google sign-in popup appears and works
- [ ] User can successfully authenticate with Google
- [ ] Application receives Google token
- [ ] Cognito Identity Pool exchanges token for AWS credentials
- [ ] Authenticated user can access private S3 objects
- [ ] Unauthenticated users cannot access private content

### STAGE 5: Cleanup the Account

#### Resource Cleanup Checklist
1. **Delete S3 Bucket**:
   - Empty bucket contents first
   - Delete the bucket

2. **Remove CloudFront Distribution**:
   - Disable distribution
   - Wait for deployment
   - Delete distribution

3. **Delete Cognito Identity Pool**:
   - This will also remove associated IAM roles

4. **Google Project Cleanup**:
   - Delete OAuth 2.0 credentials
   - Optionally delete the entire Google project

5. **Verify Cleanup**:
   - Check AWS billing for any remaining resources
   - Ensure no unexpected charges

## Key Concepts Explained

### Web Identity Federation (WEBIDF)
Web Identity Federation allows you to create AWS-backed applications without managing AWS credentials directly in your application. Instead, users authenticate with trusted external providers, and AWS provides temporary credentials based on that authentication.

### Token Exchange Flow
1. **User Authentication**: User signs in with Google (or other provider)
2. **Token Receipt**: Application receives identity token from provider
3. **Credential Exchange**: Cognito exchanges provider token for temporary AWS credentials
4. **Resource Access**: Application uses AWS credentials to access AWS services

### Security Benefits
- **No Long-term Credentials**: Application never stores permanent AWS access keys
- **Temporary Access**: Credentials expire automatically
- **Fine-grained Permissions**: Different permissions for authenticated vs unauthenticated users
- **External Identity Trust**: Leverage existing identity providers

## Best Practices

### Security
- Always use HTTPS for production applications
- Implement proper token validation
- Use least-privilege IAM policies
- Regularly rotate and review permissions

### Performance
- Cache credentials appropriately (but respect expiration)
- Use CloudFront for global application delivery
- Optimize S3 object access patterns

### Monitoring
- Enable CloudTrail for API call logging
- Monitor Cognito authentication metrics
- Set up alerts for unusual access patterns

## Troubleshooting

### Common Issues

#### Google Authentication Fails
- **Cause**: Incorrect Client ID or domain configuration
- **Solution**: Verify Google Console settings and authorized domains

#### Access Denied for Private S3 Objects
- **Cause**: IAM role permissions or identity pool configuration
- **Solution**: Check authenticated role policies and trust relationships

#### CORS Errors
- **Cause**: S3 bucket CORS configuration
- **Solution**: Configure proper CORS settings for your application domain

#### Token Exchange Fails
- **Cause**: Identity pool configuration or provider setup
- **Solution**: Verify identity provider settings in Cognito

### Debugging Steps
1. Check browser developer console for errors
2. Verify network requests and responses
3. Test IAM role permissions independently
4. Validate Google token manually

## Real-World Applications

### Use Cases
- **Media Applications**: Secure access to user-specific content
- **Document Management**: Private file sharing with authentication
- **IoT Applications**: Device-specific data access
- **Gaming**: Player-specific game assets and progress

### Scaling Considerations
- Identity pools can handle millions of identities
- Consider using Cognito User Pools for more complex user management
- Implement proper caching strategies for high-traffic applications

## Conclusion

This implementation demonstrates a complete Web Identity Federation solution using AWS Cognito Identity Pools. The architecture provides:

- **Seamless Authentication**: Users sign in with familiar providers
- **Secure AWS Access**: Temporary credentials with appropriate permissions
- **Scalable Architecture**: Handles growth from prototype to production
- **Cost-Effective**: Pay only for what you use

The demo shows how "Patches the cat" images are securely accessed only by authenticated users, illustrating the practical application of federated identity in protecting private resources while maintaining a smooth user experience.

## Next Steps

Consider extending this implementation by:
- Adding multiple identity providers (Facebook, Amazon, etc.)
- Implementing Cognito User Pools for additional user management
- Adding API Gateway with custom authorizers
- Implementing more complex IAM policies for different user types
- Adding user attribute mapping and claims transformation
