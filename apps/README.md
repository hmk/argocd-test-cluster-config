# Tailscale and Headlamp Setup

This directory contains the Argo CD applications for Tailscale and Headlamp, which work together to provide secure access to the Headlamp Kubernetes dashboard.

## Components

1. **Tailscale Operator** - Provides secure networking and ingress capabilities
2. **Headlamp** - A Kubernetes dashboard for cluster management

## Setup Instructions

### 1. Apply the Applications

The applications will be automatically deployed by Argo CD since they're included in the main kustomization.yaml.

### 2. Configure Tailscale

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