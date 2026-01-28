# ArgoCD Google OAuth Configuration

This directory contains the configuration for enabling Google OAuth authentication in ArgoCD using Dex.

## Components

1. **OnePasswordItem** (`google-oauth-secret.yaml`) - Syncs Google OAuth credentials from 1Password
2. **ConfigMap** (`argocd-cm.yaml`) - Base ArgoCD configuration with Dex connector placeholders
3. **RBAC ConfigMap** (`argocd-rbac-cm.yaml`) - Role-based access control mapping for Google users
4. **Hook Job** (`dex-secret-injector-job.yaml`) - Injects real credentials into Dex configuration
5. **RBAC** (`dex-secret-injector-rbac.yaml`) - Permissions for the hook Job

## How It Works

1. OnePassword operator syncs Google OAuth credentials to a Kubernetes Secret
2. ArgoCD applies the base ConfigMap with placeholder values
3. The hook Job runs before Dex starts and:
   - Waits for the OnePassword-created secret
   - Reads the real clientID/clientSecret values
   - Patches the ConfigMap with real values
   - Restarts the Dex deployment
4. Dex starts with the real Google OAuth configuration
5. Users can log in with Google OAuth (jake@heimark.org has admin access)

## Secret Rotation

The hook Job automatically re-runs when:
- The OnePassword secret is updated (credential rotation)
- ArgoCD syncs the application
- The checksum annotation changes

This ensures credentials are always up-to-date without manual intervention.