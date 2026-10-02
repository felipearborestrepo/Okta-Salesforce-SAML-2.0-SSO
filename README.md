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

### Step 2: Collect the IdP Values from Okta
- On the app's Sign On tab → Metadata details → More details:

| Okta value | Used in Salesforce as |
|---|---|
| Sign on URL | Identity Provider Login URL |
| Issuer | Issuer |
| Signing Certificate (Download) | Identity Provider Certificate |
| Sign out URL | Custom Logout URL |

<img width="729" height="797" alt="Image 10-1-26 at 21 01 2" src="https://github.com/user-attachments/assets/842575d6-4058-47f2-9631-77ecfc2f254d" />

### Step 3: Configure SAML in Salesforce
- Salesforce Setup → Single Sign-On Settings → enable SAML Enabled → New:

| Field | Value |
|---|---|
| Name | Okta |
| Issuer | `exk188dmjsnClBwdv698` (exactly as in Okta's metadata, see Troubleshooting) |
| Identity Provider Certificate | Certificate downloaded from Okta |
| Entity ID | `https://ruby-customer-6867.my.salesforce.com` |
| Request Signature Method | RSA-SHA256 |
| Assertion Decryption Certificate | Assertion not encrypted |
| SAML Identity Type | Assertion contains the **Federation ID** from the User object |
| SAML Identity Location | NameIdentifier element of the Subject statement |
| SP-Initiated Request Binding | HTTP POST |
| Identity Provider Login URL | Okta Sign on URL |
| Custom Logout URL | Okta Sign out URL |

**Design decision:** I matched users by **Federation ID** instead of Salesforce username. Salesforce usernames must be globally unique across all Salesforce orgs, so they often can't match an Okta username. A Federation ID decouples the two.

<img width="1624" height="866" alt="Image 10-1-26 at 21 02" src="https://github.com/user-attachments/assets/0bb86394-6d57-4eeb-af3f-5f89eb9e8da9" />

### Step 4: Exchange Metadata Back to Okta
- After saving, Salesforce generates its own Login URL (in the Endpoints section). I pasted it into the Okta app's Sign On → Login URL field. SAML trust needs values from both sides; each side tells the other where to send and receive assertions.

<img width="1477" height="862" alt="Image 10-1-26 at 21 03" src="https://github.com/user-attachments/assets/98d5f137-d838-4d25-b198-aaed2f626682" />

### Step 5: Create the Matching Salesforce User
- Created an active Salesforce user (Standard User profile) for OktaUser3.
- Set Federation ID = oktauser3@<tenant>.onmicrosoft.com, which exactly matches the user's Okta username (the value Okta sends as the SAML NameID).

<img width="1197" height="753" alt="Image 10-1-26 at 21 03 (1)" src="https://github.com/user-attachments/assets/201f7984-a902-48a8-949d-79433a09c747" />

<img width="1370" height="785" alt="Image 10-1-26 at 21 05" src="https://github.com/user-attachments/assets/edd9142f-ac9b-4cf0-88a6-b7423cd9c055" />

### Step 6: Enable Okta on the Salesforce Login Page
- Setup → My Domain → Authentication Configuration → checked Okta as an authentication service.

<img width="1221" height="868" alt="Image 10-1-26 at 21 10" src="https://github.com/user-attachments/assets/b25e40f5-fdb7-4268-8428-25f9937e4214" />

### Step 7: Assign the App in Okta
- Assigned the Salesforce app on the Assignments tab so OktaUser3 sees the Salesforce tile on their dashboard. Access is protected by an Okta MFA authentication policy (Okta Verify).

<img width="1704" height="663" alt="Image 10-1-26 at 20 57" src="https://github.com/user-attachments/assets/0e10d76d-d145-40e2-aad3-fb4fec718419" />

### Step 8: Test (IdP-Initiated SSO)
- Opened a private browser window (to avoid existing admin sessions).

- ## Troubleshooting: "Single Sign-On Error"

The first test failed with Salesforce's generic **"We can't log you in because of an issue with single sign-on."**

| Step | What I did | What I found |
|---|---|---|
| 1 | Ran Salesforce's **SAML Assertion Validator** | *Unable to load a config from the assertion's issuer and audience*. Salesforce couldn't match the assertion to any SAML config. |
| 2 | Checked the Okta app ID and **Custom Domain** (which controls the Audience) | Both were correct, so the Audience wasn't the problem. |
| 3 | Pulled Okta's IdP metadata with `curl` to read the real `entityID` | Okta sends the Issuer as just `exk188dmjsnClBwdv698`. |
| 4 | Compared it to Salesforce's config | Salesforce had `http://www.okta.com/exk188dmjsnClBwdv698`, a mismatch. |
| **Fix** | Updated the Salesforce Issuer to match the metadata exactly | SSO succeeded. |

```bash
curl -s https://<okta-domain>/app/<app-id>/sso/saml/metadata | grep -o 'entityID="[^"]*"'
```

**Lesson:** Always verify SAML values against the IdP's actual metadata instead of assuming a standard format, or import the metadata directly to avoid manual errors.
- Signed in to the Okta dashboard as OktaUser3 and approved the Okta Verify MFA prompt.
- Clicked the Salesforce.com tile.
- Landed in Salesforce as OktaUser with no Salesforce password prompt.

<img width="1701" height="848" alt="Image 10-1-26 at 21 09" src="https://github.com/user-attachments/assets/c7bf582b-f4f1-4275-a85c-59df86c831b5" />
