# Tailscale and Headlamp Setup

This directory contains the Argo CD applications for Tailscale and Headlamp, which work together to provide secure access to the Headlamp Kubernetes dashboard.

## Components

1. **Tailscale Operator** - Provides secure networking and ingress capabilities
2. **Headlamp** - A Kubernetes dashboard for cluster management

## Setup Instructions

### 1. Create Tailscale OAuth Credentials in 1Password

Before deploying, you need to set up Tailscale OAuth credentials:

1. **Create an OAuth client in Tailscale:**
   - Go to [Tailscale Admin Console > Settings > OAuth Clients](https://login.tailscale.com/admin/settings/oauth)
   - Click "Generate OAuth Client"
   - Set the following scopes:
     - `devices:core` (read/write)
     - `auth_keys` (write)
     - `services` (write)
   - Set the tag: `tag:k8s-operator`
   - Click "Generate client"
   - Copy the **Client ID** and **Client Secret** (you won't be able to see the secret again!)

2. **Update the 1Password item:**
   - Open the `tailscale-credentials` item in the `ArgoCDTest` vault in 1Password
   - Add two fields:
     - Field name: `client_id` → Value: (paste the Client ID from Tailscale)
     - Field name: `client_secret` → Value: (paste the Client Secret from Tailscale)
   - Save the item

3. **Verify the 1Password Operator is running:**
   ```bash
   kubectl -n onepassword get pods
   ```

### 2. Apply the Applications

The applications will be automatically deployed by Argo CD since they're included in the main kustomization.yaml.

The `tailscale-oauth` OnePasswordItem will automatically create a Kubernetes Secret named `operator-oauth` in the `tailscale` namespace, which the Tailscale operator will use for authentication.

### 3. Verify Tailscale Authentication

After deployment, you'll need to authenticate the Tailscale operator:

1. Get the operator's auth URL:
   ```bash
   kubectl -n tailscale get secret/operator-authurl -o go-template='{{index .data "authurl"}}' | base64 -d
   ```

2. Visit the URL to authenticate the operator with your Tailscale account.

### 3. Access Headlamp

Once both applications are running and Tailscale is configured:

1. Find your Headlamp URL in the Tailscale admin console or use:
   ```bash
   kubectl -n headlamp get ingress headlamp-tailscale-ingress
   ```

2. Access Headlamp through the Tailscale-provided MagicDNS name.

## Security Notes

- Headlamp is configured with `anonymousAccess.enable: "false"` to require authentication
- Tailscale provides secure, encrypted access to the dashboard
- Access is restricted to devices on your Tailscale network

## Troubleshooting

If you're having issues:

1. Check that both applications are synced in Argo CD
2. Verify the Tailscale operator is authenticated
3. Ensure MagicDNS is enabled in your Tailscale admin console
4. Check that the ingress is properly created:
   ```bash
   kubectl -n headlamp describe ingress headlamp-tailscale-ingress
   ```