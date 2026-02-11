# tcloudpublic-functiongraph-iam-idp-key-rotation
FunctiohGraph script to automatically rotate OIDC IdP signing keys


# Manual Deployment Guide

Step-by-step guide to deploy the IDP Key Rotation function using  Console.

# Required Access

You need an  account with these permissions:
- IAM: Create policies, create agencies, attach policies
- FunctionGraph: Create and configure functions

# Required Information

 Step 1: Create IAM Policy

In  Console, click Identity and Access Management. In left sidebar, click Permissions -> Policies/Roles, click Create Custom Policy button

Switch to JSON tab
Enter this content:

Policy Name: IDP-Key-Rotation-Policy
```json
{
  "Version": "1.1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:identityProviders:listIdentityProviders",
        "iam:identityProviders:getIdentityProvider",
        "iam:identityProviders:updateIdentityProvider",
        "iam:identityProviders:getProtocol",
        "iam:identityProviders:updateProtocol",
        "iam:identityProviders:getOpenIDConnectConfig",
        "iam:identityProviders:updateOpenIDConnectConfig"
      ]
    }
  ]
}
```
Save Policy

 Step 2: Create Agency
In IAM console (from Step 1), click Agencies in left sidebar, click Create Agency button

Fill in the agency form:

| Field | Value | Notes |
|-------|-------|-------|
| Agency Name | `idp-key-rotation-agency` | Unique name for this agency |
| Agency Type | Cloud service | Agency for services |
| Cloud Service | FunctionGraph | Service that will use this agency |
| Validity Period | Unlimited | Agency doesn't expire |
| Description | "Agency for IDP key rotation function" | Optional but recommended |

Click Next, then select the IDP-Key-Rotation-Policy policy created in Step 1
Scope should show "All resources"

Click OK, then Finish


## Step 1: Gather Configuration Values

Before starting, collect these values:

| Variable | Where to Find | Your Value |
|----------|---------------|------------|
| `DOMAIN_NAME` | Console → My Credentials → Domain Name | |
| `PROJECT_NAME` | Usually `eu-de` | |
| `IDP_IDS` | IAM → Identity Providers (exact names) | |
| `AZURE_TENANT_ID` | Azure Portal → Entra ID → Tenant ID | |

## Step 2: Create IDP in OTC IAM (Azure)

1. Log into Console
2. Navigate to IAM (Identity and Access Management)
3. Click Identity Providers in left sidebar
4. Click Create Identity Provider

**Fill in the form:**

| Field | Value | Example | Notes |
|-------|-------|---------|-------|
| Name | | AzureOIDC | Case-sensitive, must match `IDP_IDS` env var |
| Protocol | OpenID Connect | OpenID Connect | |
| SSO Type | Virtual user | Virtual user | |
| Status | Enabled | Enabled | |
| Description | Descriptive text | `Azure AD OIDC authentication` | Optional |

5. Click OK, then click Modify Identity Provider

**Fill in OIDC configuration:**

| Field | Value | Example | Notes |
|-------|-------|---------|-------|
| Access Type | programmatic access and management console | |  |
| Identity Provider URL | `https://login.microsoftonline.com/{AZURE_TENANT_ID}/v2.0` | `https://login.microsoftonline.com/4606102c-e3b4-4bba-a38f-cf14aa589558/v2.0` | From Azure tenant ID |
| Client ID | Application (client) ID from Azure | `12345678-1234-1234-...` | From Azure app registration |
| Authorization endpoint | `https://login.microsoftonline.com/{AZURE_TENANT_ID}/oauth2/v2.0/authorize` | `https://login.microsoftonline.com/4606102c-e3b4-4bba-a38f-cf14aa589558/oauth2/v2.0/authorize` | |
| Scopes | openid email profile | openid email profile | |
| Response Type | `id_token` | `id_token` | |
| Response Mode | `form_post` | `form_post` | |
| Signing Key | paste the content from https://login.microsoftonline.com/common/discovery/v2.0/keys | |  |

## Step 3: Create IDP in OTC IAM (Google)

1. Log into OTC Console
2. Navigate to IAM (Identity and Access Management)
3. Click Identity Providers in left sidebar
4. Click Create Identity Provider

**Fill in the form:**

| Field | Value | Example | Notes |
|-------|-------|---------|-------|
| Name | | GoogleOIDC | Case-sensitive, must match `IDP_IDS` env var |
| Protocol | OpenID Connect | OpenID Connect | |
| SSO Type | Virtual user | Virtual user | |
| Status | Enabled | Enabled | |
| Description | Descriptive text | `Google OIDC authentication` | Optional |

5. Click OK, then click Modify Identity Provider

**Fill in OIDC configuration:**

| Field | Value | Example | Notes |
|-------|-------|---------|-------|
| Access Type | programmatic access and management console | |  |
| Identity Provider URL | `https://accounts.google.com` | `https://accounts.google.com` |  |
| Client ID | Application (client) ID from Google | `654307938795-940laq83d985vnk3fngh8d3obrkrpuus.apps.googleusercontent.com` |  |
| Authorization endpoint | `https://accounts.google.com/o/oauth2/v2/auth` | `https://accounts.google.com/o/oauth2/v2/auth` | |
| Scopes | openid email profile | openid email profile | |
| Response Type | `id_token` | `id_token` | |
| Response Mode | `form_post` | `form_post` | |
| Signing Key | paste the content from https://www.googleapis.com/oauth2/v3/certs | |  |

## Step 4: Configure Identity Conversion Rules (Mapping)

1. After protocol creation, go to Identity Conversion Rules tab
2. Click Create Rule or Set Rule

**Mapping configuration:**
```json
[
  {
    "remote": [
      {
        "type": "email"
      }
    ],
    "local": [
      {
        "user": {
          "name": "{0}"
        }
      }
    ]
  }
]
```

## Step 5: Import ZIP Package

1. Download attached idp-key-rotation_latest.zip
2. Go To FunctionGraph - Functions - Function List - Import Function
3. Select Downloaded zip and Import

## Step 6: Create Function in Console

### 6.1 Navigate to FunctionGraph

1. Log in to Console: https://console.otc.t-systems.com/
2. Select region: eu-de
3. Go to: Service List → Compute → FunctionGraph
4. Click Functions in the left menu

### 6.2 Create New Function

1. Click Create Function
2. Fill in the form:

| Field | Value |
|-------|-------|
| Function Name | `idp-key-rotation` |
| Agency | Select existing or create new with FunctionGraph permissions |
| Runtime | Python 3.9 |
| Handler | `index.handler` |
| Code Entry Type | Upload ZIP |
| Upload | Click and select your `function.zip` |

3. Click Create Function

## Step 7: Configure Function Settings

### 7.1 Basic Settings

After creation, click on the function name to open settings.

1. Go to Configuration tab
2. Set:

| Setting | Value |
|---------|-------|
| Memory | 256 MB |
| Timeout | 60 seconds |

3. Click Save

### 7.2 Environment Variables

1. Go to Configuration → Environment Variables
2. Click Add Environment Variable for each:

| Key | Value |
|-----|-------|
| `DOMAIN_NAME` | Your domain name (from Step 1) |
| `PROJECT_NAME` | `eu-de` |
| `IDP_IDS` | `AzureOIDC,GoogleOIDC` (or your IDP names) |
| `AZURE_TENANT_ID` | Your Azure tenant ID (if using Azure) |

3. Click Save

## Step 8: Test the Function

### 8.1 Test Authentication

1. Go to the function page
2. Click Test tab
3. In the test event input, enter:

```json
{"action": "test_auth"}
```

4. Click Test
5. Check the result - should show:

```json
{
  "statusCode": 200,
  "body": {
    "message": "OTC authentication successful",
    "token_preview": "MIIGaQYJKoZIhvcNAQcC..."
  }
}
```

## Step 9: Create Timer Trigger (Scheduled Execution)

1. Go to function page → Triggers tab
2. Click Create Trigger
3. Select Timer

### 9.1 Configure Timer

| Field | Value |
|-------|-------|
| Name | `daily-key-rotation` |
| Trigger Status | Enabled |
| Schedule Type | Cron |
| Cron Expression | `0 0 0 * * ?` |
| Additional Information | `{"action": "rotate"}` |

### 10 Check results

After running, the script should have changed the config of the IDP, changing the description of affected IDPs to "Key rotated via FunctionGraph at XXX"
