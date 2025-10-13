# Single Sign-On (SSO) Integration

## Overview
Single Sign-On integration specifications for seamless authentication across organizational systems.

## Description
Comprehensive SSO implementation framework supporting SAML, OAuth 2.0, and OpenID Connect protocols for integration with school systems, identity providers, and enterprise authentication solutions.

## Purpose

SSO integration enables MLC-LMS to authenticate users through external identity providers (IDPs), allowing:

- **Seamless School Integration**: Students and teachers log in using existing school credentials (no separate username/password)
- **Reduced Administrative Burden**: Automatic user provisioning and role assignment based on IDP attributes
- **Enhanced Security**: Leverage enterprise-grade authentication systems with centralized access control
- **Compliance**: Meet district IT requirements for centralized identity management
- **Improved User Experience**: Single set of credentials across all educational platforms

SSO is particularly valuable for Ensemble subscriptions serving schools, districts, and educational organizations with existing identity infrastructure.

## Scope

**Included:**
- SAML 2.0 Service Provider (SP) implementation
- OAuth 2.0 / OpenID Connect (OIDC) Relying Party implementation
- Identity provider configuration and metadata exchange
- Attribute mapping and role assignment rules
- Just-in-time (JIT) user provisioning
- Account linking for existing users
- SSO configuration UI for Subscriber-Admins
- Multi-tenant SSO (different IDP per organization)
- Common IDP integrations (Google Workspace, Microsoft 365, ClassLink, Clever)
- SSO error handling and fallback authentication
- SSO session management and logout flows

**Excluded:**
- Internal authentication flows (see [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
- MLC-native authentication (username/password) for non-SSO users
- Security architecture details (see [Security and Privacy](../18-architecture/security-privacy.md))
- Role definitions and permissions (see [Roles Matrix](../02-roles-and-permissions/roles-matrix.md))
- System admin impersonation (see [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md))

## Supported SSO Protocols

### SAML 2.0

**Protocol Overview:**
- Industry-standard XML-based protocol for enterprise SSO
- MLC-LMS acts as Service Provider (SP), school/district acts as Identity Provider (IDP)
- Supports SP-initiated and IDP-initiated authentication flows
- Used primarily by K-12 school districts and enterprise customers

**Key Components:**
- **SP Metadata**: MLC-LMS metadata XML describing service endpoints and certificates
- **IDP Metadata**: Customer IDP metadata describing authentication endpoints and certificates
- **Assertions**: Signed XML tokens containing user attributes (name, email, role)
- **Certificates**: X.509 certificates for signature verification and encryption

**SAML Flows:**

1. **SP-Initiated Flow** (User starts at MLC-LMS):
   - User visits MLC-LMS login page
   - User selects their organization or enters email domain
   - MLC-LMS generates SAML AuthnRequest
   - User is redirected to IDP login page
   - User authenticates at IDP
   - IDP generates signed SAML Response with user attributes
   - User is redirected back to MLC-LMS with SAML Response
   - MLC-LMS validates signature and creates user session

2. **IDP-Initiated Flow** (User starts at school portal):
   - User authenticates at school portal/dashboard
   - User clicks MLC-LMS link in portal
   - IDP generates signed SAML Response
   - User is redirected to MLC-LMS with SAML Response
   - MLC-LMS validates signature and creates user session

**SAML Endpoints:**
- **Single Sign-On (SSO) URL**: `https://musiclearningcommunity.com/api/saml/sso`
- **Assertion Consumer Service (ACS) URL**: `https://musiclearningcommunity.com/api/saml/acs`
- **Single Logout (SLO) URL**: `https://musiclearningcommunity.com/api/saml/slo`
- **SP Metadata URL**: `https://musiclearningcommunity.com/api/saml/metadata`

### OAuth 2.0 / OpenID Connect (OIDC)

**Protocol Overview:**
- Modern token-based authentication protocol
- OAuth 2.0 for authorization, OIDC for authentication layer on top
- Used primarily by Google Workspace, Microsoft 365, and modern IDPs
- Simpler implementation than SAML, JSON-based instead of XML

**Key Components:**
- **Authorization Endpoint**: IDP URL where users authenticate
- **Token Endpoint**: IDP URL where MLC-LMS exchanges authorization code for tokens
- **ID Token**: JWT containing user identity claims (sub, email, name)
- **Access Token**: Token used to fetch additional user info from IDP
- **UserInfo Endpoint**: IDP URL providing user profile data

**OIDC Flow (Authorization Code Flow):**
1. User visits MLC-LMS login page
2. User selects their organization or identity provider
3. MLC-LMS redirects user to IDP authorization endpoint
4. User authenticates at IDP and grants consent
5. IDP redirects user back to MLC-LMS with authorization code
6. MLC-LMS exchanges authorization code for ID token and access token
7. MLC-LMS validates ID token signature and creates user session
8. (Optional) MLC-LMS fetches additional user attributes from UserInfo endpoint

**OIDC Endpoints:**
- **Callback URL**: `https://musiclearningcommunity.com/api/oidc/callback`
- **Post-Logout Redirect URL**: `https://musiclearningcommunity.com/api/oidc/logout`

## Identity Provider Configuration

### Organization-Level SSO Setup

**Configuration Steps:**
1. **Enable SSO for Organization**: Subscriber-Admin enables SSO in organization settings
2. **Select Protocol**: Choose SAML 2.0 or OIDC based on IDP capabilities
3. **Exchange Metadata**: Upload IDP metadata XML (SAML) or enter OIDC endpoints
4. **Configure Attribute Mapping**: Map IDP attributes to MLC roles and profile fields
5. **Test Connection**: Validate SSO configuration with test login
6. **Enable for Users**: Activate SSO for all users in organization

**Configuration Data Model:**

```typescript
interface SSOConfiguration {
  id: string;
  organizationId: string;
  protocol: "saml" | "oidc";
  enabled: boolean;
  enforced: boolean; // If true, disable username/password auth
  
  // SAML-specific
  samlIdpEntityId?: string;
  samlIdpSsoUrl?: string;
  samlIdpSloUrl?: string;
  samlIdpCertificate?: string;
  samlSignRequests?: boolean;
  samlWantAssertionsSigned?: boolean;
  
  // OIDC-specific
  oidcIssuer?: string;
  oidcAuthorizationUrl?: string;
  oidcTokenUrl?: string;
  oidcUserInfoUrl?: string;
  oidcClientId?: string;
  oidcClientSecret?: string; // Encrypted
  oidcScopes?: string[]; // e.g., ["openid", "profile", "email"]
  
  // Attribute mapping
  attributeMapping: AttributeMapping;
  
  // JIT provisioning
  jitProvisioningEnabled: boolean;
  jitDefaultRole: "teacher" | "student";
  
  createdAt: Date;
  updatedAt: Date;
}

interface AttributeMapping {
  // Identity attributes
  externalUserId: string; // IDP attribute name for unique user ID
  email: string; // IDP attribute name for email
  firstName: string; // IDP attribute name for first name
  lastName: string; // IDP attribute name for last name
  
  // Role mapping
  roleAttribute: string; // IDP attribute name containing role
  roleMapping: RoleMapping[]; // Map IDP role values to MLC roles
}

interface RoleMapping {
  idpRoleValue: string; // Value from IDP (e.g., "teacher", "staff")
  mlcRole: "subscriber-admin" | "teacher-admin" | "teacher" | "student";
}
```

### Pre-Configured IDP Templates

**Google Workspace for Education:**
- **Protocol**: OIDC
- **Pre-configured endpoints**: Google OIDC discovery URL
- **Attribute mapping**: Standard Google claims (sub, email, given_name, family_name)
- **Setup steps**: Admin provides Google Workspace domain, creates OAuth client

**Microsoft 365 / Azure AD:**
- **Protocol**: SAML 2.0 or OIDC (both supported)
- **Pre-configured endpoints**: Microsoft Azure AD endpoints
- **Attribute mapping**: Standard Azure AD claims
- **Setup steps**: Admin creates app registration in Azure AD, provides tenant ID

**ClassLink:**
- **Protocol**: SAML 2.0
- **Pre-configured endpoints**: ClassLink SAML IDP
- **Attribute mapping**: ClassLink standard attributes (email, role, school)
- **Setup steps**: Admin adds MLC-LMS as ClassLink application

**Clever:**
- **Protocol**: OAuth 2.0 with Clever API
- **Pre-configured endpoints**: Clever OAuth endpoints
- **Attribute mapping**: Clever role types (teacher, student, school_admin)
- **Setup steps**: Admin creates Clever application, provides client ID/secret

### Custom IDP Configuration

For organizations with custom identity providers (Active Directory, Okta, Shibboleth, etc.):

**SAML Configuration Requirements:**
- IDP Entity ID
- IDP Single Sign-On URL
- IDP X.509 Certificate (for signature verification)
- (Optional) IDP Single Logout URL

**OIDC Configuration Requirements:**
- Issuer URL (or Discovery Document URL)
- Authorization Endpoint URL
- Token Endpoint URL
- (Optional) UserInfo Endpoint URL
- Client ID (provided by IDP)
- Client Secret (provided by IDP, stored encrypted)

## Attribute Mapping and Role Assignment

### Standard Attributes

**Required Attributes:**
- **External User ID**: Unique identifier from IDP (SAML `NameID`, OIDC `sub` claim)
- **Email**: User's email address (used for notifications and account recovery)
- **First Name**: User's first name
- **Last Name**: User's last name

**Optional Attributes:**
- **Role**: User's role in school system (teacher, student, admin)
- **Grade Level**: Student's grade level (for automatic placement)
- **School ID**: School identifier for multi-school districts
- **Class/Section ID**: Class roster information
- **Student ID**: School's internal student identifier

### Role Mapping Rules

**Mapping Strategy:**

Organizations configure mappings from IDP role values to MLC roles:

```typescript
// Example role mapping for school district
const roleMapping: RoleMapping[] = [
  { idpRoleValue: "district_admin", mlcRole: "subscriber-admin" },
  { idpRoleValue: "school_admin", mlcRole: "teacher-admin" },
  { idpRoleValue: "teacher", mlcRole: "teacher" },
  { idpRoleValue: "staff", mlcRole: "teacher" },
  { idpRoleValue: "student", mlcRole: "student" },
];
```

**Fallback Behavior:**
- If IDP role attribute is missing or doesn't match mapping, use `jitDefaultRole` (configured per organization)
- If no default role configured, deny SSO login and show error message
- Subscriber-Admin can manually assign role after initial SSO attempt

**Multiple Roles:**
- If IDP provides multiple roles (array), MLC uses the highest-privilege role
- Privilege order: subscriber-admin > teacher-admin > teacher > student

### Custom Attribute Mapping

**Flexible Attribute Mapping:**

Subscriber-Admins can specify which IDP attribute corresponds to each MLC field:

```typescript
// Example: Google Workspace custom attributes
const attributeMapping: AttributeMapping = {
  externalUserId: "sub", // Google user ID
  email: "email",
  firstName: "given_name",
  lastName: "family_name",
  roleAttribute: "hd", // Use hosted domain to determine role
  roleMapping: [
    { idpRoleValue: "teachers.district.edu", mlcRole: "teacher" },
    { idpRoleValue: "students.district.edu", mlcRole: "student" },
  ],
};

// Example: Azure AD with custom claims
const attributeMapping: AttributeMapping = {
  externalUserId: "oid", // Azure object ID
  email: "upn", // User Principal Name
  firstName: "given_name",
  lastName: "family_name",
  roleAttribute: "extension_MLC_Role", // Custom Azure AD attribute
  roleMapping: [
    { idpRoleValue: "Music Teacher", mlcRole: "teacher" },
    { idpRoleValue: "Student", mlcRole: "student" },
  ],
};
```

## Just-In-Time (JIT) Provisioning

### JIT User Creation

**Automatic User Provisioning:**

When JIT provisioning is enabled, MLC-LMS automatically creates user accounts on first SSO login:

1. **User Attempts SSO Login**: User logs in via IDP for first time
2. **Validate SSO Response**: MLC-LMS validates SAML assertion or OIDC token
3. **Check for Existing User**: Look up user by external user ID
4. **If User Not Found**:
   - Create new user account with attributes from IDP
   - Assign role based on role mapping rules
   - Link user to organization
   - Create initial user profile
5. **Create Session**: Log user in with new account

**Configuration Options:**
- **JIT Enabled**: Boolean flag to enable/disable JIT provisioning
- **JIT Default Role**: Fallback role if role mapping fails (teacher or student)
- **JIT Create Teachers**: Allow automatic teacher account creation
- **JIT Create Students**: Allow automatic student account creation
- **JIT Require Manual Approval**: Created accounts require Subscriber-Admin approval before activation

### JIT User Updates

**Attribute Synchronization:**

On each SSO login, MLC-LMS can update user attributes from IDP:

- **Always Updated**: Email, first name, last name (keep in sync with IDP)
- **Conditionally Updated**: Role (only if role mapping result changes)
- **Never Updated**: Username, user-generated content (profile settings, preferences)

**Configuration:**
- **Sync Attributes on Login**: Boolean flag to enable attribute updates
- **Sync Role Changes**: Allow role changes based on IDP updates (handle with care)

## Account Linking

### Linking Existing Accounts

**Scenario:** Organization enables SSO, but users already have MLC username/password accounts.

**Account Linking Process:**

**Option 1: Email-Based Automatic Linking**
1. User attempts SSO login
2. MLC-LMS receives email from IDP
3. Check if user with matching email exists
4. If found, automatically link SSO identity to existing account
5. User can now log in via SSO or username/password

**Option 2: Manual Linking Flow**
1. User attempts SSO login
2. MLC-LMS detects matching email but account not linked
3. Show "Link Account" screen
4. User enters existing MLC password to confirm ownership
5. Link SSO identity to existing account
6. User can now log in via SSO or username/password

**Configuration:**
- **Auto-Link by Email**: Enable automatic linking for matching emails
- **Require Password Confirmation**: Require password verification for security
- **Allow Multiple IDPs**: User can link multiple SSO identities to one account

### Account Linking Data Model

```typescript
interface UserSSOIdentity {
  id: string;
  userId: string; // MLC user ID
  ssoConfigId: string; // Which SSO configuration
  externalUserId: string; // IDP user ID (SAML NameID or OIDC sub)
  idpEmailAddress: string; // Email from IDP
  linkedAt: Date;
  lastLoginAt: Date;
}
```

## SSO Configuration UI

### Subscriber-Admin SSO Setup Wizard

**Step 1: Enable SSO**
- Toggle "Enable Single Sign-On for this organization"
- Warning: "Users will be able to log in using your organization's identity provider"

**Step 2: Choose Identity Provider**
- **Pre-configured options**: Google Workspace, Microsoft 365, ClassLink, Clever
- **Custom SAML provider**: Upload IDP metadata XML
- **Custom OIDC provider**: Enter OIDC endpoints manually

**Step 3: Configure Attribute Mapping**
- Visual mapping interface: IDP attribute → MLC field
- Role mapping table: Add rules to map IDP roles to MLC roles
- Default role dropdown: Fallback role for unmapped users

**Step 4: Configure Provisioning**
- JIT provisioning toggle
- Auto-link existing accounts toggle
- Require manual approval for new accounts toggle

**Step 5: Download SP Metadata**
- Download MLC-LMS SAML metadata XML
- Copy ACS URL and Entity ID for manual IDP configuration
- Copy OIDC Callback URL for OAuth configuration

**Step 6: Test SSO**
- "Test SSO Login" button opens new tab with SSO flow
- Shows detailed debug info (SAML assertion, OIDC token claims)
- Validates configuration before enabling for all users

**Step 7: Enable for Users**
- "Enable SSO for all users" button
- (Optional) "Enforce SSO" toggle to disable username/password auth

### SSO Management Dashboard

**For Subscriber-Admins:**

**SSO Status Panel:**
- SSO enabled/disabled status
- Identity provider name
- Last successful login timestamp
- Total users logged in via SSO (last 30 days)

**User Account Management:**
- List users with SSO linked accounts
- Show external user ID and last SSO login
- Manually link/unlink SSO identities
- View SSO login history per user

**SSO Logs:**
- Recent SSO authentication attempts (success/failure)
- Error messages for failed SSO logins
- SAML/OIDC debug information (for troubleshooting)

## Multi-Tenant SSO

### Organization-Specific IDPs

**Architecture:**
- Each organization (subscription) can configure its own SSO provider
- Multiple organizations can use the same IDP (e.g., all Google Workspace)
- Single IDP can serve multiple MLC organizations (district with multiple schools)

**Organization Discovery:**

**Option 1: Email Domain Lookup**
- User enters email address on login page
- MLC-LMS looks up organization by email domain
- If SSO configured, redirect to IDP
- If not, show username/password form

**Option 2: Organization Selector**
- Login page shows "Login with SSO" button
- User enters organization name or selects from list
- MLC-LMS redirects to organization's IDP

**Option 3: Direct SSO URL**
- Organization has dedicated SSO login URL
- Example: `https://musiclearningcommunity.com/sso/springfield-schools`
- URL maps to organization, skips discovery step

### Multi-School Districts

**Scenario:** School district with multiple schools, each school is separate Ensemble subscription.

**Shared IDP Configuration:**
- District configures single SSO connection
- All schools (organizations) share same IDP
- Use school ID attribute to route users to correct organization
- JIT provisioning creates user in correct organization based on school ID

**School Attribute Mapping:**
```typescript
interface DistrictSSOConfiguration extends SSOConfiguration {
  schoolIdAttribute: string; // IDP attribute containing school ID
  schoolMapping: SchoolMapping[]; // Map school IDs to organizations
}

interface SchoolMapping {
  idpSchoolId: string; // School ID from IDP
  mlcOrganizationId: string; // MLC organization (subscription)
}
```

## SSO Error Handling

### Common Error Scenarios

**SAML Errors:**

| Error | Cause | User Message | Resolution |
|-------|-------|--------------|------------|
| `InvalidSignature` | SAML assertion signature invalid | "SSO authentication failed. Please contact your administrator." | Verify IDP certificate is correct and not expired |
| `ExpiredAssertion` | SAML assertion timestamp too old | "SSO session expired. Please try again." | Check clock sync between IDP and MLC-LMS |
| `InvalidAudience` | Assertion audience doesn't match SP Entity ID | "SSO configuration error. Please contact your administrator." | Verify SP Entity ID matches in IDP configuration |
| `MissingAttribute` | Required attribute not in assertion | "Your account is missing required information. Please contact your administrator." | Add required attributes to IDP attribute release |
| `RoleMappingFailed` | IDP role doesn't match any mappings | "Your account role is not recognized. Please contact your administrator." | Add role mapping or configure default role |

**OIDC Errors:**

| Error | Cause | User Message | Resolution |
|-------|-------|--------------|------------|
| `InvalidToken` | ID token signature invalid or expired | "SSO authentication failed. Please try again." | Verify OIDC client ID/secret and token endpoint |
| `InvalidIssuer` | Token issuer doesn't match configuration | "SSO configuration error. Please contact your administrator." | Verify OIDC issuer URL is correct |
| `InsufficientScope` | Required OIDC scopes not granted | "Additional permissions are required. Please contact your administrator." | Request additional scopes (email, profile) from IDP |
| `AccessDenied` | User denied consent at IDP | "You must grant permission to continue." | User must accept consent screen at IDP |

### Fallback Authentication

**When SSO Fails:**

1. **Show Error Message**: Display user-friendly error message
2. **Offer Username/Password Login**: "Having trouble? [Login with username and password](#)"
3. **Contact Support**: "Need help? Contact your organization administrator."

**Configuration:**
- **Allow Fallback**: If SSO enforced, don't show username/password option
- **Error Logging**: Log detailed error for admin troubleshooting
- **Admin Notifications**: Alert Subscriber-Admin of repeated SSO failures

## SSO Session Management

### Session Lifecycle

**SSO Login:**
1. User authenticates at IDP
2. MLC-LMS creates session (see [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
3. Session includes SSO identity information
4. Session timeout follows standard MLC-LMS rules

**SSO Logout:**

**Single Logout (SLO) Support:**
- When user logs out of MLC-LMS, optionally log them out of IDP
- When user logs out of IDP, optionally log them out of MLC-LMS
- Requires IDP support for SAML SLO or OIDC RP-initiated logout

**Logout Flows:**

**Option 1: Local Logout Only** (Default)
- User clicks logout in MLC-LMS
- MLC-LMS terminates local session
- User remains logged in at IDP (can SSO back without re-authenticating)

**Option 2: IDP Logout (SAML SLO)**
- User clicks logout in MLC-LMS
- MLC-LMS sends SAML LogoutRequest to IDP
- IDP logs user out and sends LogoutResponse
- MLC-LMS terminates local session

**Option 3: IDP Logout (OIDC)**
- User clicks logout in MLC-LMS
- MLC-LMS redirects user to IDP end_session_endpoint
- IDP logs user out and redirects back to MLC-LMS
- MLC-LMS terminates local session

**Configuration:**
- **SLO Enabled**: Enable single logout to IDP
- **SLO Required**: Require IDP logout (don't allow local-only logout)

### Session Security

**SSO Session Considerations:**
- SSO sessions follow same timeout rules as native authentication
- MFA at IDP counts as MFA for MLC-LMS (no second MFA required)
- Impersonation by System Admin works normally for SSO users
- Account lockout doesn't apply to SSO users (managed by IDP)

## Common IDP Integration Guides

### Google Workspace Integration

**Setup Steps:**

1. **Create OAuth Client in Google Admin**
   - Go to Google Cloud Console > APIs & Credentials
   - Create OAuth 2.0 Client ID
   - Application type: Web application
   - Authorized redirect URIs: `https://musiclearningcommunity.com/api/oidc/callback`

2. **Configure MLC-LMS**
   - Select "Google Workspace" from IDP templates
   - Enter OAuth Client ID and Client Secret
   - Configure attribute mapping (usually defaults work)
   - Set role mapping based on Google Workspace OUs or groups

3. **Test Integration**
   - Use "Test SSO" to validate configuration
   - Verify user attributes are mapped correctly
   - Check role assignment works as expected

**Google-Specific Features:**
- Automatic email domain verification
- Google Classroom integration (future)
- OAuth incremental authorization for additional scopes

### Microsoft 365 / Azure AD Integration

**Setup Steps:**

1. **Create App Registration in Azure AD**
   - Go to Azure Portal > Azure Active Directory > App Registrations
   - New registration: MLC-LMS SSO
   - Supported account types: Accounts in this organizational directory only
   - Redirect URI: `https://musiclearningcommunity.com/api/saml/acs` (SAML) or `https://musiclearningcommunity.com/api/oidc/callback` (OIDC)

2. **Configure App in Azure AD**
   - Certificates & Secrets: Create client secret (copy immediately)
   - API Permissions: Add Microsoft Graph permissions (User.Read, email, profile)
   - Enterprise Applications: Configure user assignment and attributes

3. **Download Metadata and Configure MLC-LMS**
   - For SAML: Download Federation Metadata XML from Azure
   - Upload metadata to MLC-LMS SSO configuration
   - Configure attribute mapping (Azure AD claims)
   - Set role mapping based on Azure AD groups or directory roles

**Azure AD-Specific Features:**
- Support for Azure AD groups-based role assignment
- Conditional Access policies (MFA, device compliance)
- Azure AD B2B for guest users (future)

### ClassLink Integration

**Setup Steps:**

1. **Add MLC-LMS in ClassLink**
   - Log in to ClassLink Administrator Dashboard
   - Applications > Add Application > Custom SAML 2.0
   - Enter MLC-LMS SAML metadata URL or upload SP metadata XML

2. **Configure Attribute Release**
   - Release required attributes: email, firstName, lastName, role
   - ClassLink uses standard attribute names

3. **Configure MLC-LMS**
   - Select "ClassLink" from IDP templates
   - Upload ClassLink IDP metadata
   - Configure role mapping (ClassLink role values: teacher, student, admin)

**ClassLink-Specific Features:**
- Rostering data integration (future)
- ClassLink Analytics integration
- Mobile app SSO support

### Clever Integration

**Setup Steps:**

1. **Create Clever Application**
   - Log in to Clever Developer Dashboard
   - Create new application
   - Select OAuth 2.0 authentication
   - Enter redirect URI: `https://musiclearningcommunity.com/api/clever/callback`

2. **Configure MLC-LMS**
   - Select "Clever" from IDP templates
   - Enter Clever Client ID and Client Secret
   - Configure Clever API scopes

3. **Configure Clever Data Sync** (Optional)
   - Enable Clever rostering data sync
   - Map Clever sections to MLC classes
   - Automatically assign students to teachers based on Clever roster

**Clever-Specific Features:**
- Clever Secure Sync for roster data
- Clever Instant Login (one-click SSO)
- Clever Library integration

## Telemetry

### SSO Events

Track SSO-specific events via [Event Model](../15-analytics-and-reporting/event-model.md):

- `sso.login.initiated` - SSO login flow started
- `sso.login.success` - Successful SSO authentication
- `sso.login.failure` - SSO authentication failed
- `sso.logout.initiated` - SSO logout initiated
- `sso.logout.success` - Successful SSO logout (with SLO)
- `sso.config.created` - SSO configuration created
- `sso.config.updated` - SSO configuration modified
- `sso.config.deleted` - SSO configuration removed
- `sso.config.tested` - SSO test connection performed
- `sso.account.linked` - User account linked to SSO identity
- `sso.account.unlinked` - User account unlinked from SSO identity
- `sso.jit.user_created` - New user created via JIT provisioning
- `sso.jit.user_updated` - User attributes updated via JIT sync
- `sso.error.invalid_signature` - SAML signature validation failed
- `sso.error.invalid_token` - OIDC token validation failed
- `sso.error.role_mapping_failed` - Role mapping failed, no default role
- `sso.error.attribute_missing` - Required attribute missing from IDP

**Event Properties:**
- `organization_id`: Organization using SSO
- `sso_config_id`: SSO configuration ID
- `protocol`: saml | oidc
- `idp_name`: Identity provider name (Google, Azure AD, custom)
- `external_user_id`: IDP user identifier
- `role_assigned`: MLC role assigned after SSO
- `jit_provisioning`: Boolean, was user auto-created
- `error_code`: Error code for failed SSO attempts
- `error_message`: Human-readable error message

## Permissions

### SSO Configuration Management

Based on [Roles Matrix](../02-roles-and-permissions/roles-matrix.md):

- **Subscriber-Admin**: Full SSO configuration management for their organization
  - Enable/disable SSO
  - Configure IDP settings and metadata
  - Manage attribute mappings and role assignments
  - View SSO logs and user linkages
  - Test SSO configuration
  
- **Teacher-Admin**: View SSO status and linked users (read-only)
  - See which users have SSO enabled
  - View SSO login history
  - Cannot modify SSO configuration

- **Teacher**: No SSO configuration access

- **Student**: No SSO configuration access

- **System Admin**: Global SSO management across all organizations
  - View and edit all SSO configurations
  - Troubleshoot SSO issues for any organization
  - Access detailed SSO logs and debug info
  - Manually link/unlink user accounts

### SSO Login Permissions

- **All Users**: Can use SSO if configured for their organization
- **Enforced SSO**: If SSO is enforced, username/password login disabled for users
- **Fallback Auth**: System Admin can always use username/password (bypass SSO enforcement)

## Supporting Documents Referenced

This SSO integration specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role hierarchy and user types that must be supported through SSO role mapping and JIT provisioning
- [Blayzer note USER ROLES.docx.txt](../00-ORIG-CONTEXT/Blayzer%20note%20USER%20ROLES.docx.txt) — Additional role requirements and organizational structure for Ensemble subscriptions
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator requirements for managing user accounts and authentication across organizations
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional system admin specifications for user profile management and organizational hierarchy views

## Dependencies

### Internal Dependencies
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Session management after SSO authentication
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions for SSO role mapping
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Permissions assigned to SSO-provisioned users
- [Security and Privacy](../18-architecture/security-privacy.md) - Security architecture and encryption for SSO
- [Event Model](../15-analytics-and-reporting/event-model.md) - SSO telemetry events
- [Config and Secrets](../18-architecture/config-and-secrets.md) - Secure storage of SSO credentials and certificates

### External Dependencies
- **Identity Provider (IDP)**: Customer's authentication system (Google, Microsoft, etc.)
- **SAML Library**: Node.js SAML library (e.g., `passport-saml`, `saml2-js`)
- **OIDC Library**: OpenID Connect library (e.g., `openid-client`, `passport-oidc`)
- **Certificate Management**: X.509 certificate handling for SAML signatures
- **JWT Validation**: JSON Web Token validation for OIDC ID tokens
- **XML Processing**: XML parsing and validation for SAML assertions

### IDP-Specific Dependencies
- **Google Workspace**: Google OAuth 2.0 / OIDC endpoints
- **Microsoft 365**: Azure AD SAML/OIDC endpoints
- **ClassLink**: ClassLink SAML IDP and rostering API
- **Clever**: Clever OAuth 2.0 API and rostering data
- **Custom IDPs**: SAML 2.0 or OIDC compliant identity providers

## Open Questions

### SSO Implementation

- Should we support SAML artifact binding or only HTTP POST/Redirect?
- What level of SAML encryption should we support (encrypted assertions)?
- Should we implement OIDC dynamic client registration?
- How should we handle IDP certificate rotation for SAML?

### User Provisioning

- Should we support automatic de-provisioning when users are removed from IDP?
- How should we handle user attribute conflicts during JIT sync (e.g., email change)?
- Should we support multi-IDP accounts (user links multiple SSO identities)?
- What happens to user data when SSO account is unlinked?

### Role Management

- Should we support group-based role assignment (IDP groups → MLC roles)?
- How should we handle temporary role changes (e.g., student becomes teacher)?
- Should we allow manual role override for SSO-provisioned users?
- How do we handle users with roles at multiple schools in same district?

### District/Multi-School Scenarios

- Should we support district-level SSO with automatic school assignment?
- How should we handle students/teachers moving between schools?
- Should we support cross-school teacher assignments for music specialists?
- What level of organizational hierarchy should we support?

### Advanced Features

- Should we support SCIM for automated user provisioning/deprovisioning?
- Should we implement Just-in-Time (JIT) group/class provisioning?
- How should we handle SSO for mobile apps (OAuth PKCE flow)?
- Should we support social login (personal Google/Microsoft accounts) for home users?

### Compliance and Security

- What level of audit logging is required for SSO events?
- Should we support SAML request signing in addition to assertion validation?
- How should we handle GDPR right-to-erasure for SSO-linked accounts?
- What certificate lifetimes and rotation policies should we enforce?