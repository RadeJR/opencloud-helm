# External OIDC Provider Configuration

This guide explains how to configure OpenCloud to use external OIDC (OpenID Connect) providers for authentication instead of the built-in Keycloak instance.

## Overview

OpenCloud supports three authentication modes:

1. **Built-in Keycloak** (default) - Deploys and configures Keycloak automatically
2. **External OIDC Provider** - Use Auth0, Okta, Azure AD, Google, or any OIDC-compliant provider
3. **Internal IDP** - Use OpenCloud's built-in identity provider (when both Keycloak and external OIDC are disabled)

> **Note:** When using external OIDC or Keycloak, OpenCloud's internal IDP service is automatically disabled. If you disable both, the internal IDP will be used for authentication.

## Quick Start

### Using Built-in Keycloak (Default)

```yaml
keycloak:
  internal:
    enabled: true  # This is the default
```

### Using External OIDC Provider

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://auth.example.com"
    clientId: "opencloud-client"

keycloak:
  internal:
    enabled: false
```

### Using Internal IDP (No External Authentication)

```yaml
global:
  oidc:
    enabled: false

keycloak:
  internal:
    enabled: false  # OpenCloud's internal IDP will be used
```

## Configuration Options

### Required Settings

When `global.oidc.enabled: true`, the following settings are **required**:

| Setting | Description | Example |
|---------|-------------|---------|
| `global.oidc.issuer` | Base URL of your OIDC provider | `https://auth.example.com` |
| `global.oidc.clientId` | Client/Application ID from your OIDC provider | `opencloud-client` |

### Optional Settings

| Setting | Description | Default | Example |
|---------|-------------|---------|---------|
| `global.oidc.accountUrl` | URL for user account management | (none) | `https://auth.example.com/account` |
| `global.oidc.scopes` | OAuth scopes to request | `openid profile email groups roles` | `openid email profile` |
| `global.oidc.userClaim` | Token claim for user identification | `preferred_username` | `email` |
| `global.oidc.roleClaim` | Token claim containing user roles | `roles` | `groups` |
| `global.oidc.autoProvision` | Auto-create users on first login | `true` | `false` |
| `global.oidc.roleDriver` | Role assignment method | `oidc` | `default` |

## Provider-Specific Examples

### Auth0

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://yourdomain.auth0.com"
    clientId: "your-client-id"
    accountUrl: "https://yourdomain.auth0.com/account"
    userClaim: "email"
    roleClaim: "https://yourdomain.com/roles"  # Custom namespace

keycloak:
  internal:
    enabled: false
```

**Auth0 Setup:**
1. Create a Regular Web Application in Auth0 Dashboard
2. Add OpenCloud URL to Allowed Callback URLs
3. Configure custom claims in Auth0 Rules/Actions if needed

### Okta

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://yourorg.okta.com/oauth2/default"
    clientId: "your-client-id"
    accountUrl: "https://yourorg.okta.com/enduser/settings"
    roleClaim: "groups"

keycloak:
  internal:
    enabled: false
```

**Okta Setup:**
1. Create an OIDC Web Application in Okta Admin Console
2. Add OpenCloud redirect URIs
3. Add `groups` claim to ID token in authorization server

### Azure AD (Microsoft Entra ID)

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://login.microsoftonline.com/{tenant-id}/v2.0"
    clientId: "your-application-id"
    accountUrl: "https://myaccount.microsoft.com"
    roleClaim: "roles"

keycloak:
  internal:
    enabled: false
```

**Azure AD Setup:**
1. Register an application in Azure Portal
2. Configure Web platform with redirect URI
3. Add optional claims for roles in Token Configuration

### Google

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://accounts.google.com"
    clientId: "your-client-id.apps.googleusercontent.com"
    accountUrl: "https://myaccount.google.com"
    userClaim: "email"
    roleDriver: "default"  # Google doesn't provide role claims

keycloak:
  internal:
    enabled: false
```

**Google Setup:**
1. Create OAuth 2.0 Client ID in Google Cloud Console
2. Add authorized redirect URIs
3. Note: Role-based access requires custom implementation

### GitLab

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://gitlab.com"
    clientId: "your-application-id"
    accountUrl: "https://gitlab.com/-/profile"
    userClaim: "email"
    roleClaim: "groups"

keycloak:
  internal:
    enabled: false
```

**GitLab Setup:**
1. Create an Application in GitLab User Settings > Applications
2. Select `openid`, `profile`, `email` scopes
3. Add redirect URI

### External Keycloak

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://keycloak.example.com/realms/myrealm"
    clientId: "opencloud"
    accountUrl: "https://keycloak.example.com/realms/myrealm/account"

keycloak:
  internal:
    enabled: false
```

## Installation

### Option 1: Using Custom Values File

```bash
# Create a values file for your provider
cat > my-oidc-config.yaml <<EOF
global:
  oidc:
    enabled: true
    issuer: "https://auth.example.com"
    clientId: "opencloud"

keycloak:
  internal:
    enabled: false
EOF

# Install with custom values
helm install opencloud . -f my-oidc-config.yaml
```

### Option 2: Using --set Flags

```bash
helm install opencloud . \
  --set global.oidc.enabled=true \
  --set global.oidc.issuer=https://auth.example.com \
  --set global.oidc.clientId=opencloud \
  --set keycloak.internal.enabled=false
```

### Option 3: Using Example Files

```bash
# Copy and customize an example
cp examples/external-oidc-providers.yaml my-config.yaml
# Edit my-config.yaml with your provider details

# Install
helm install opencloud . -f my-config.yaml
```

## Configuration Hierarchy

The configuration follows this precedence order:

1. **External OIDC** (`global.oidc.enabled: true`) - Takes highest priority
2. **Internal Keycloak** (`keycloak.internal.enabled: true`) - Used if external OIDC not enabled
3. **OpenCloud defaults** - Used when specific settings are not provided

Example demonstrating overrides:

```yaml
global:
  oidc:
    enabled: true
    issuer: "https://auth.example.com"
    userClaim: "email"  # Overrides default "preferred_username"

opencloud:
  appConfig:
    proxy:
      oidc:
        userOidcClaim: "preferred_username"  # This is overridden by global.oidc.userClaim
```

## Troubleshooting

### Error: "global.oidc.issuer is required"

**Cause:** External OIDC is enabled but no issuer URL provided.

**Solution:**
```yaml
global:
  oidc:
    enabled: true
    issuer: "https://your-provider.com"  # Add this
```

### Users Cannot Login

**Check:**
1. Client ID matches the one registered with your OIDC provider
2. Redirect URI is configured in your OIDC provider
3. Required scopes (`openid profile email`) are granted
4. Token claims contain expected values

**Debug:**
```bash
# View rendered environment variables
helm template opencloud . -f my-config.yaml \
  --show-only templates/opencloud/deployment.yaml | grep -A 100 "env:"
```

### No Account Management Link

**Cause:** `accountUrl` is not configured.

**Solution:**
```yaml
global:
  oidc:
    accountUrl: "https://your-provider.com/account"
```

### Roles Not Working

**Check:**
1. `roleClaim` matches the claim name in your token
2. OIDC provider includes role/group claims in tokens
3. `roleDriver` is set to `oidc` (not `default`)

**Azure AD Example:**
```yaml
global:
  oidc:
    roleClaim: "roles"  # Must match token claim
    roleDriver: "oidc"
```

### Token Verification Failures

**Check:**
```yaml
opencloud:
  appConfig:
    proxy:
      oidc:
        accessTokenVerifyMethod: "jwt"  # or "none" for debugging
```

## Advanced Configuration

### Custom Token Claims

Some providers use custom claim namespaces:

```yaml
global:
  oidc:
    userClaim: "https://example.com/username"
    roleClaim: "https://example.com/roles"
```

### Disable Auto-Provisioning

Require manual user creation:

```yaml
global:
  oidc:
    autoProvision: false
```

### Static Role Assignment

Use default roles instead of OIDC claims:

```yaml
global:
  oidc:
    roleDriver: "default"
```

### Fine-Grained Control

Override any setting in `opencloud.appConfig`:

```yaml
opencloud:
  appConfig:
    proxy:
      oidc:
        rewriteWellknown: false
        accessTokenVerifyMethod: "none"
        userCs3Claim: "email"
```

## Security Considerations

1. **Use HTTPS:** Always use HTTPS for OIDC issuer URLs
2. **Client Secrets:** Store client secrets in Kubernetes secrets
3. **Redirect URIs:** Restrict redirect URIs in your OIDC provider
4. **Token Validation:** Keep `accessTokenVerifyMethod: "jwt"` in production
5. **Scopes:** Only request necessary scopes
6. **Auto-Provisioning:** Disable if you need manual user approval

## Migration from Built-in Keycloak

To migrate from built-in Keycloak to external OIDC:

1. **Export Users:** Export user data from Keycloak
2. **Configure External Provider:** Set up users in external OIDC provider
3. **Update Helm Values:** Switch to external OIDC configuration
4. **Test:** Verify authentication works with test users
5. **Deploy:** Upgrade Helm release with new configuration

```bash
# Upgrade to external OIDC
helm upgrade opencloud . -f external-oidc.yaml
```

## Reference

### Environment Variables Generated

When OIDC is configured, these environment variables are set:

- `OC_OIDC_ISSUER` - OIDC provider base URL
- `WEB_OIDC_CLIENT_ID` - OAuth client ID
- `WEB_OIDC_METADATA_URL` - OIDC discovery endpoint
- `WEB_OIDC_SCOPE` - OAuth scopes
- `WEB_OPTION_ACCOUNT_EDIT_LINK_HREF` - Account management URL
- `PROXY_USER_OIDC_CLAIM` - User identification claim
- `PROXY_ROLE_ASSIGNMENT_OIDC_CLAIM` - Role claim
- `PROXY_AUTOPROVISION_ACCOUNTS` - Auto-provisioning flag
- `PROXY_ROLE_ASSIGNMENT_DRIVER` - Role driver type

### Complete Example

See [examples/external-oidc-providers.yaml](examples/external-oidc-providers.yaml) for comprehensive examples of all supported providers.

## Support

For issues or questions:
- Check the OpenCloud documentation
- Review OIDC provider documentation
- Verify token claims using JWT debugging tools
- Enable debug logging: `opencloud.appConfig.log.level: "debug"`
