# AWS SSO (Identity Center) Implementation Guide

## Project Overview

This project demonstrates how to implement AWS Single Sign-On (SSO), now called AWS Identity Center, to centralize user access management across multiple AWS accounts within an organization. The implementation allows users to access multiple AWS accounts with a single set of credentials while maintaining granular permission control.

## Architecture Overview

![SSO Architecture](image1.png)

The architecture shows:
- **SSO User Portal**: Central authentication point for users
- **AWS Single Sign-On (SSO)**: Core identity management service
- **SSO Identity Source**: Manages users and groups
- **A4L AWS Organization**: Contains multiple AWS accounts (General/Org Mgmt, Prod, Dev)
- **Centralized Permissions Management**: Permission sets applied across accounts
- **User Access Control**: Different permission levels for different users

## What You'll Learn

- How to set up AWS Identity Center (SSO)
- Create and configure permission sets
- Create users and groups in Identity Center
- Assign users to groups with specific permissions
- Configure AWS account access through permission sets
- Test single sign-on functionality

## Prerequisites

- AWS Organization set up with multiple accounts
- Administrative access to the AWS Organization management account
- Basic understanding of AWS IAM concepts

## Step-by-Step Implementation

### Step 1: Enable AWS Identity Center

1. Navigate to AWS Identity Center in the AWS Management Console
2. Enable Identity Center for your organization
3. Choose your identity source (AWS Identity Center directory recommended for this demo)

### Step 2: Create Permission Sets

Permission sets define what level of access users will have when they assume roles in AWS accounts.

![Permission Sets](image3.png)

1. In AWS Identity Center, navigate to **Permission sets**
2. Click **Create permission set**
3. Create the following permission sets:

#### Permission Set 1: AdministratorAccess
- **Name**: `AdministratorAccess`
- **Description**: Full administrative access to AWS resources
- **Policies**: Attach AWS managed policy `AdministratorAccess`

#### Permission Set 2: ViewOnlyAccess
- **Name**: `ViewOnlyAccess`
- **Description**: Read-only access to AWS resources
- **Policies**: Attach AWS managed policy `ViewOnlyAccess`

#### Permission Set 3: Billing
- **Name**: `Billing`
- **Description**: Access to billing and cost management
- **Policies**: Attach AWS managed policy `Billing`

#### Permission Set 4: PowerUserAccess
- **Name**: `PowerUserAccess`
- **Description**: Power user access (everything except user management)
- **Policies**: Attach AWS managed policy `PowerUserAccess`

### Step 3: Create Users

1. In AWS Identity Center, navigate to **Users**
2. Click **Add user**
3. Create a user named **Sally**:
   - **Username**: `sally`
   - **Email**: `sally@example.com` (use your test email)
   - **First name**: `Sally`
   - **Last name**: `TestUser`
   - **Display name**: `Sally TestUser`
4. Set a temporary password or send an email invitation

### Step 4: Create Groups

1. In AWS Identity Center, navigate to **Groups**
2. Click **Create group**
3. Create a group named **Billing**:
   - **Group name**: `Billing`
   - **Description**: `Users with billing access`

### Step 5: Add Users to Groups

1. Navigate to the **Billing** group you just created
2. Click **Add users**
3. Select **Sally** and add her to the Billing group

### Step 6: Configure AWS Account Access

1. Navigate to **AWS accounts** in Identity Center
2. Select your AWS account(s)
3. Click **Assign users or groups**
4. Choose **Groups** and select the **Billing** group
5. Select the **Billing** permission set
6. Click **Submit**

This configuration means:
- **Sally** (in the Billing group) gets **billing only access** to the specified AWS account(s)
- **IAMADMIN** (in the PowerUsers group) gets **full account control**

### Step 7: Test SSO Access

![AWS Access Portal](image4.png)

1. Open the AWS Access Portal URL (provided in Identity Center dashboard)
2. Sign in as Sally using her credentials
3. You should see the available AWS accounts she has access to
4. Click on an account to assume the role with billing permissions

![Billing Console](image2.png)

5. Verify that Sally can only access billing-related services and information
6. Test that she cannot access other AWS services due to permission restrictions

## Key Features Demonstrated

### Centralized Permission Management
- Permission sets are created once and can be applied across multiple AWS accounts
- Changes to permissions are automatically synchronized across all assigned accounts

### User and Group Management
- Users are organized into logical groups based on their roles
- Groups can be assigned different permission sets for different accounts

### Single Sign-On Experience
- Users authenticate once and can access multiple AWS accounts
- No need to manage separate credentials for each AWS account

### Granular Access Control
- Different users get different levels of access based on their group membership
- Permission sets ensure consistent access patterns across accounts

## Security Benefits

1. **Centralized Identity Management**: All user identities managed in one place
2. **Consistent Permissions**: Permission sets ensure uniform access across accounts
3. **Reduced Credential Sprawl**: Single set of credentials for multiple accounts
4. **Audit Trail**: Centralized logging of all access attempts
5. **Temporary Credentials**: Uses temporary credentials for enhanced security

## Best Practices

1. **Use Groups**: Always assign permissions to groups, not individual users
2. **Principle of Least Privilege**: Grant only the minimum necessary permissions
3. **Regular Review**: Periodically review and audit permission assignments
4. **Clear Naming**: Use descriptive names for permission sets and groups
5. **Documentation**: Document the purpose and scope of each permission set

## Troubleshooting

### User Cannot Access AWS Account
- Verify user is in the correct group
- Check that the group has been assigned to the AWS account
- Confirm the correct permission set is assigned

### Permission Denied Errors
- Review the permission set policies
- Ensure the permission set includes necessary permissions for the required actions
- Check for any restrictive policies that might be blocking access

### SSO Portal Not Loading
- Verify the SSO service is enabled
- Check the access portal URL
- Ensure the user account is active and not suspended

## Conclusion

This implementation demonstrates a complete AWS SSO (Identity Center) setup that provides:
- Centralized user management across multiple AWS accounts
- Granular permission control through permission sets
- Simplified user experience with single sign-on
- Enhanced security through temporary credentials and centralized audit trails

The example shows how Sally, a billing user, can access billing information across multiple AWS accounts while being restricted from other services, demonstrating the power of centralized identity and access management in AWS.

## Next Steps

Consider expanding this implementation by:
- Integrating with external identity providers (Active Directory, Azure AD)
- Creating more specialized permission sets for different roles
- Implementing automated user provisioning
- Setting up advanced monitoring and alerting for access patterns
