# AWS-Professional-Projects

# Project: Single Sign On
**Date:** July 2025  
**Tech Stack:**  SAML 2.0 
**Status:** Completed

<img width="1202" height="657" alt="Screenshot 2025-07-12 231947" src="https://github.com/user-attachments/assets/2f83c7db-df2a-4698-b311-2433c418c363" />

🛡️ AWS Identity Center (SSO) - Centralized Permission Management Project
This project demonstrates how to configure AWS Identity Center (formerly AWS Single Sign-On) to centrally manage access to multiple AWS accounts using permission sets, users, and groups.

📌 Project Objective
To provide secure, role-based access to AWS accounts across an organization by:

Creating Identity Center permission sets

Creating a user (Sally) and a group (Billing)

Assigning permissions to the group

Linking access to all AWS accounts in the organization

✅ Step-by-Step Guide
1️⃣ Create Permission Sets
In the IAM Identity Center Console:

Navigate to: AWS Console > IAM Identity Center > Permission sets

Click Create permission set

Create the following permission sets:

Billing – for billing-only access

AdministratorAccess – full access

PowerUserAccess – all services except IAM and Organizations

ViewOnlyAccess – read-only access

📸 Screenshot:
<img width="2186" height="483" alt="Screenshot 2025-07-12 235903" src="https://github.com/user-attachments/assets/b72a08df-153f-437c-a1b7-7b124796885d" />


2️⃣ Create a User (Sally)
Go to IAM Identity Center > Users

Click Add user

Enter:

Name: Sally

Email: your test email

Leave the rest as default or configure password settings as needed

3️⃣ Create a Group and Add Sally
Navigate to IAM Identity Center > Groups

Click Create group

Name: Billing

Add Sally to the group

4️⃣ Assign Group to AWS Accounts
Go to AWS Accounts section under Identity Center

Select All accounts (e.g., Development, Production, General)

Assign the Billing group with the Billing permission set

5️⃣ Test Access via the AWS Access Portal
Log in as Sally using the Identity Center link provided

You’ll see the assigned accounts (Development, Production, General)

Each account will show Billing access only

📸 Screenshot:
<img width="2467" height="666" alt="Screenshot 2025-07-13 000933" src="https://github.com/user-attachments/assets/92d34a0b-2758-4c33-988c-a9e43060709e" />

<img width="2491" height="897" alt="Screenshot 2025-07-13 001205" src="https://github.com/user-attachments/assets/81f61a3b-0330-43fe-8be6-7d85cc42b1a8" />

💡 Summary
This project implements:

Centralized access control using AWS Identity Center

Role-based access with Permission Sets

Group-based user management

Seamless account switching for users via the AWS Access Portal
