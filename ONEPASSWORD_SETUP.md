# 1Password Setup Instructions

## Prerequisites
Before deploying the 1Password Connect server, you need to set up the credentials in your Kubernetes cluster.

## Step 1: Create 1Password Connect Credentials

1. In your 1Password account, create a new Connect server:
   - Go to 1Password.com > Settings > Business > Workflow > Secrets Automation
   - Click "Create Connect Server"
   - Give it a name like "k3s-cluster"
   - Download the credentials file (`1password-credentials.json`)

2. Save the credentials file to `.env` (for repeatability):
   ```bash
   # Copy the credentials file to .env for local reference
   cp /path/to/1password-credentials.json .env/1password-credentials.json

   # Make sure .env is in .gitignore (it should be!)
   echo ".env/" >> .gitignore
   ```

3. Create the Kubernetes secret from the credentials file:
   ```bash
   # The namespace will be auto-created by ArgoCD due to CreateNamespace=true
   kubectl create secret generic op-credentials \
     --from-file=1password-credentials.json=.env/1password-credentials.json \
     --namespace onepassword
   ```

   Or using base64 (alternative method):
   ```bash
   # Base64 encode the credentials file
   base64 -i .env/1password-credentials.json

   # Then create a secret manifest (don't commit this!)
   # Or apply directly with kubectl create secret generic
   ```

## Step 2: Verify Deployment

After setting up the credentials, the 1Password Connect server and Operator should be deployed automatically by ArgoCD.

Check the status:
```bash
# Check if pods are running
kubectl -n onepassword get pods

# Check if the test app is working
kubectl -n onepassword-test get onepassworditems
kubectl -n onepassword-test get secrets
```

## Step 3: Create Test Item in 1Password

1. Create a new item in your 1Password vault:
   - Vault: Create or use an existing vault (e.g., "k8s-test")
   - Item name: "test-item"
   - Add a field:
     - Name: "test-key"
     - Value: "test-value"

2. Update the `onepassworditem.yaml` with the correct vault and item names if needed.