
# **Integrating Secrets with Doppler using Helm on Ubuntu**

This guide walks you through the process of integrating Doppler secrets with your Kubernetes cluster using Helm on an Ubuntu machine. You'll install necessary tools, authenticate Doppler, and deploy a Helm chart that integrates with Doppler to manage secrets.

## **Prerequisites**
Before you begin, ensure the following:
- An Ubuntu machine (local or remote).
- A Kubernetes cluster set up and configured (`kubectl` installed).
- Helm installed on your Ubuntu machine.
- A Doppler account and a configured project.

## **Step 1: Install Helm**

Helm is a package manager for Kubernetes that allows you to define, install, and upgrade complex Kubernetes applications.

1. **Update the package list:**
   ```bash
   sudo apt-get update
   ```

2. **Install Helm:**
   Use the following command to download and install Helm:
   ```bash
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```

3. **Verify Helm Installation:**
   Confirm Helm is installed correctly:
   ```bash
   helm version
   ```
   This should display the version of Helm you have installed.

## **Step 2: Install Doppler CLI**

The Doppler CLI is used to interact with the Doppler platform, manage secrets, and perform various operations.

1. **Install the Doppler CLI:**
   ```bash
   curl -Ls https://cli.doppler.com/install.sh | sudo sh
   ```

2. **Verify Doppler Installation:**
   Check if Doppler CLI is installed by running:
   ```bash
   doppler --version
   ```
   This should display the version of the Doppler CLI installed.

## **Step 3: Authenticate Doppler CLI**

After installing the Doppler CLI, you need to authenticate with your Doppler account to access your secrets.

1. **Login to Doppler:**
   ```bash
   doppler login
   ```
   This will prompt you to authenticate via your Doppler account.

2. **Setup Doppler Project and Environment:**
   ```bash
   doppler setup
   ```
   Follow the prompts to select your project and environment within Doppler.

## **Step 4: Install the Doppler Helm Plugin**

To integrate Doppler secrets with Helm, you'll need to install the Doppler Helm plugin.

1. **Add Doppler Helm Plugin:**
   ```bash
   helm plugin install https://github.com/DopplerHQ/helm-doppler
   ```

2. **Verify Plugin Installation:**
   List installed Helm plugins to verify:
   ```bash
   helm plugin list
   ```
   You should see `doppler` listed as one of the plugins.

## **Step 5: Create a Helm Values File**

A `values.yaml` file is used to configure Helm charts. You'll specify the Doppler integration details in this file.

1. **Create `values.yaml`:**
   ```yaml
   doppler:
     serviceToken: <DOPPLER_SERVICE_TOKEN>
     config: <DOPPLER_CONFIG>
     project: <DOPPLER_PROJECT>
     secrets:
       - name: <SECRET_NAME>
         path: <SECRET_PATH>
         mountPath: <MOUNT_PATH>
   ```
   Replace the placeholders:
   - **`DOPPLER_SERVICE_TOKEN`**: Your Doppler project's service token.
   - **`DOPPLER_CONFIG`**: The environment configuration in Doppler (e.g., dev, prod).
   - **`DOPPLER_PROJECT`**: The name of your Doppler project.
   - **`SECRET_NAME`**: The name you want to give to the Kubernetes secret.
   - **`SECRET_PATH`**: The path within Doppler where the secret is stored.
   - **`MOUNT_PATH`**: The location in your pod where the secret will be mounted.

## **Step 6: Deploy with Helm**

Now that you have your `values.yaml` configured, you can deploy your Helm chart with Doppler integration.

1. **Install or Upgrade Helm Release:**
   ```bash
   helm upgrade --install <release_name> <chart_name> -f values.yaml
   ```
   Replace:
   - **`<release_name>`**: The name for your Helm release.
   - **`<chart_name>`**: The Helm chart you are deploying.

2. **Verify Deployment:**
   Check if your pods are running with the following command:
   ```bash
   kubectl get pods
   ```
   Ensure all the pods are up and running correctly.

## **Step 7: Verify Doppler Integration**

To ensure that your Doppler secrets are correctly integrated and accessible:

1. **Check Pod Logs:**
   View logs to confirm the secrets are injected properly:
   ```bash
   kubectl logs <pod_name>
   ```
   Replace **`<pod_name>`** with the name of your pod.

2. **Access Secrets within the Pod (Optional):**
   If you need to verify the secrets directly:
   ```bash
   kubectl exec -it <pod_name> -- /bin/bash
   cd <MOUNT_PATH>
   cat <SECRET_FILE>
   ```
   Replace:
   - **`<pod_name>`**: The name of your pod.
   - **`<MOUNT_PATH>`**: The mount path you specified in `values.yaml`.
   - **`<SECRET_FILE>`**: The file containing the secret.

## **Additional Tips**
- **RBAC Permissions**: Ensure your Kubernetes cluster has the necessary RBAC (Role-Based Access Control) permissions if you encounter permission issues.
- **Updating Secrets**: You can update secrets directly in Doppler, and they will be automatically synced to your Kubernetes pods without requiring redeployment.

## **Conclusion**
You have successfully integrated Doppler secrets with your Kubernetes cluster using Helm. This setup ensures secure and efficient management of secrets within your Kubernetes applications.
