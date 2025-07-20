# AWS Cognito: User/Identity Pools and Web Identity Federation Guide

## Project Overview

This project demonstrates how to implement AWS Cognito User Pools and Identity Pools with Web Identity Federation (WEBIDF) to enable secure authentication and authorization for web applications. The implementation shows how users can authenticate with external identity providers (like Google) and gain temporary AWS credentials to access AWS resources.

## Architecture Overview

<img width="1200" height="671" alt="image" src="https://github.com/user-attachments/assets/80746cce-cb16-48b5-bfa9-431e861d1782" />


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

<img width="1203" height="675" alt="image" src="https://github.com/user-attachments/assets/3d97e7ea-4579-4e63-8aac-e334a22e367e" />


**User Pools**:
- Handle user authentication and return JWT tokens
- Support social sign-in with external providers
- Provide user directory management, profiles, sign-up & sign-in
- Include MFA and other security features
- Tokens can be used for API access via Lambda Custom Authorizers or API Gateway

<img width="1194" height="675" alt="image" src="https://github.com/user-attachments/assets/3907d1ce-1f96-4f61-aabe-5dd061b38f22" />


**Identity Pools**:
- Provide access to temporary AWS credentials
- Support both authenticated and unauthenticated roles
- Work with external identity providers or Cognito User Pools
- Enable direct access to AWS services with proper IAM permissions

<img width="1203" height="683" alt="image" src="https://github.com/user-attachments/assets/0859e4ec-4a2b-4f62-8b3e-ee849dbc46dc" />


**Combined Usage**:
- User Pools handle authentication and social sign-in
- Identity Pools provide AWS credentials for authenticated users
- Applications can standardize on User Pool tokens while accessing AWS resources

## Advanced Implementation Flow

<img width="1203" height="676" alt="image" src="https://github.com/user-attachments/assets/79d8631b-44ba-4b18-86fe-61e4094a62ad" />


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

1. **Create Application Files**:

**index.html**:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>PetIDF Demo</title>
    <meta name="author" content="acantril">
    <meta name="google-signin-scope" content="profile email">
    <meta name="referrer" content="strict-origin-when-cross-origin">
    <script src="https://sdk.amazonaws.com/js/aws-sdk-2.2.19.min.js"></script>
    <script src="scripts.js"></script>
    <script src="https://accounts.google.com/gsi/client" async defer></script>
  </head>
  <body>

    <div id="g_id_onload"
         data-client_id="REPLACE_ME_GOOGLE_APP_CLIENT_ID"
         data-context="signin"
         data-ux_mode="popup"
         data-callback="onSignIn"
         data-auto_prompt="false">
    </div>
    
    <div class="g_id_signin" data-type="standard"></div>
    <p />
    <div id="viewer"></div>
    <div id="output"></div>
    
  </body>
</html>
```

**scripts.js**:
```javascript
function onSignIn(googleToken) {
  // Google have OK'd the sign-in
  // pass the token into our web app
  credentialExchange(googleToken);
}

function credentialExchange(googleToken) {
  // Create a decoded version of the token so we can print things out
  console.log("Creating decoded token...");
  const googleTokenDecoded = parseJwt(googleToken.credential);
  
  // Output some details onto the browser console to show the token working
  console.log("ID: " + googleTokenDecoded.sub);
  console.log('Full Name: ' + googleTokenDecoded.name);
  console.log("Email: " + googleTokenDecoded.email);
  
  if (googleTokenDecoded['sub']) {
    
    // We can't access anything in AWS with a google token...
    // ... so we need to exchange it using Cognito for AWS credentials
    console.log("Exchanging Google Token for AWS credentials...");
    AWS.config.region = 'us-east-1'; 
    AWS.config.credentials = new AWS.CognitoIdentityCredentials({
      IdentityPoolId: 'REPLACE_ME_COGNITO_IDENTITY_POOL_ID', // MAKE SURE YOU REPLACE THIS
      Logins: {
        'accounts.google.com': googleToken.credential
      }
    });

    // Now lets obtain the credentials we just swapped
    AWS.config.credentials.get(function(err) {
      if (!err) {
        console.log('Exchanged to Cognito Identity Id: ' + AWS.config.credentials.identityId);
        // if we are here, things are working as they should...
        // ... now lets call a function to access images, generate signed URL's and display
        accessImages();
      } else {
        // if we are here, bad things have happened, so we should error.
        document.getElementById('output').innerHTML = "<b>YOU ARE NOT AUTHORISED TO QUERY AWS!</b>";
        console.log('ERROR: ' + err);
      }
    });

  } else {
    console.log('User not logged in!');
  }
}

function accessImages() {
  
  // Using the temp AWS Credentials, lets connect to S3
  console.log("Creating Session to S3...");
  var s3 = new AWS.S3();
  var params = {
    Bucket: "REPLACE_ME_NAME_OF_PATCHES_PRIVATE_BUCKET" // MAKE SURE YOU REPLACE THIS
  }; 

  // If we are here, things are going well, lets list all of the objects in the bucket
  s3.listObjects(params, function(err, data) {
    console.log("Listing objects in patchesprivate bucket...");
    if (err) {
      document.getElementById('output').innerHTML = "<b>YOU ARE NOT AUTHORISED TO QUERY AWS!</b>";
      console.log(err, err.stack);
    } else {
      console.log('AWS response:');
      console.log(data);
      var href = this.request.httpRequest.endpoint.href;
      var bucketUrl = href + data.Name + '/';
      
      // for all of the images in the bucket, we need to generate a signedURL for the object
      var photos = data.Contents.map(function(photo) {
        var photoKey = photo.Key;
        
        console.log("Generating signedURL for : " + photoKey);
        var url = s3.getSignedUrl ('getObject', {
          Bucket: data.Name,
          Key: photoKey
        })

        var photoUrl = bucketUrl + encodeURIComponent(photoKey);
        return getHtml([
          '<span>',
            '<div>',
              '<br/>',
              '<a href="' + url + '" target="_blank"><img style="width:224px;height:224px;" src="' + url + '"/></a>',
            '</div>',
            '<div>',
              '<span>',
              '</span>',
            '</div>',
          '</span>',
        ]);
      });

      // let's take those signedURL's, create a HTML page, and display it in the web browser
      var htmlTemplate = [ '<div>',   getHtml(photos), '</div>']
      console.log("Creating and returning html...")
      document.getElementById('viewer').innerHTML = getHtml(htmlTemplate);
    }    

  });
}

// A utility function to create HTML.
function getHtml(template) {
  return template.join('\n');
}

// A utility function to decode the google token
function parseJwt(token) {
  var base64Url = token.split('.')[1];
  var base64 = base64Url.replace('-', '+').replace('_', '/');
  var plain_token = JSON.parse(window.atob(base64));
  return plain_token;
};
```

2. **Update Configuration Values**:
   Before deploying, you must replace the following placeholder values:
   - **In index.html**: Replace `REPLACE_ME_GOOGLE_APP_CLIENT_ID` with your Google Client ID from Stage 2
   - **In scripts.js**: Replace `REPLACE_ME_COGNITO_IDENTITY_POOL_ID` with your Cognito Identity Pool ID
   - **In scripts.js**: Replace `REPLACE_ME_NAME_OF_PATCHES_PRIVATE_BUCKET` with your S3 bucket name

3. **Deploy Application Files**:
   - Upload both `index.html` and `scripts.js` to your S3 bucket
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

## Code Walkthrough

### Application Flow Analysis

The demo application consists of two main files that work together to demonstrate Web Identity Federation:

#### index.html Structure
- **Google Sign-In Integration**: Uses Google's GSI (Google Sign-In) library with popup mode
- **AWS SDK**: Includes AWS SDK for JavaScript v2 for credential exchange
- **UI Elements**: Simple interface with sign-in button and content viewers

#### scripts.js Functions

**1. onSignIn(googleToken)**
- Entry point when Google authentication succeeds
- Receives the Google JWT token and passes it to credential exchange

**2. credentialExchange(googleToken)**
- Decodes the Google JWT to extract user information (ID, name, email)
- Exchanges Google token for AWS temporary credentials using Cognito Identity Pool
- Handles the core Web Identity Federation process

**3. accessImages()**
- Uses AWS credentials to connect to S3
- Lists objects in the private bucket
- Generates signed URLs for each image
- Dynamically creates HTML to display the images

**4. Utility Functions**
- `parseJwt()`: Decodes JWT tokens to readable JSON
- `getHtml()`: Helps create HTML templates

### Security Model Demonstration
The application perfectly demonstrates the security model:
- **Unauthenticated**: Cannot access private S3 objects
- **Authenticated**: Google sign-in provides temporary AWS credentials
- **Signed URLs**: Even authenticated users get time-limited access to specific objects

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
