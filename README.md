# Okta + Salesforce SAML 2.0 Single Sign-On Lab
A hands-on lab configuring SAML 2.0 single sign-on between Okta (Identity Provider) and Salesforce (Service Provider). Users authenticate once in Okta with MFA, click the Salesforce tile, and land in Salesforce without ever entering a Salesforce password.

## Project Goal
- Configure Okta as the Identity Provider for Salesforce using SAML 2.0
- Establish trust between both systems by exchanging metadata (Issuer, certificate, SSO URL, Entity ID)
- Match Okta users to Salesforce users using a Federation ID
- Test the full sign-in flow as an end user and troubleshoot any failures

### Step 1: Add the Salesforce App in Okta
- Okta Admin Console → Applications → Browse App Catalog → Salesforce.com → Add Integration.
- Instance Type: Production.
- Custom Domain: ruby-customer-6867 (the Salesforce My Domain prefix). Okta uses this to build the SAML Audience, which must match the Salesforce Entity ID.
- Sign-on method: SAML 2.0 (not Secure Web Authentication, which only fills in passwords and isn't federation).
- Application username format: Okta username.

<img width="1709" height="875" alt="Image 10-1-26 at 21 01" src="https://github.com/user-attachments/assets/b4abfdaa-c786-4c64-81a9-d88249e43816" />

<img width="1186" height="819" alt="Image 10-1-26 at 21 11" src="https://github.com/user-attachments/assets/f82584ab-6dc2-4073-813d-6fb57e553f68" />

| Okta value | Used in Salesforce as |
|---|---|
| Sign on URL | Identity Provider Login URL |
| Issuer | Issuer |
| Signing Certificate (Download) | Identity Provider Certificate |
| Sign out URL | Custom Logout URL |

