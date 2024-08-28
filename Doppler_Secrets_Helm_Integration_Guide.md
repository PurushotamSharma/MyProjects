
# Doppler Integration with Helm and Kubernetes

## Introduction

This document provides a step-by-step guide to integrating Doppler with Kubernetes using Helm. The Doppler Kubernetes Operator will manage your secrets in Kubernetes, syncing them automatically from Doppler.

## Pre-requisites

Before starting, ensure you have the following:

- **Ubuntu Machine**
- **Helm** installed. If not, install it [here](https://helm.sh/docs/intro/install/).
- **Doppler CLI** installed. Install it using the [official guide](https://docs.doppler.com/docs/install-cli).
- **kubectl** installed and configured to interact with your Kubernetes cluster.

## Installation Steps

### 1. Add Doppler Helm Repository

Add the Doppler Helm repository to your Helm installation:

```bash
helm repo add doppler https://helm.doppler.com
helm repo update
```

### 2. Install the Doppler Kubernetes Operator

Install the Doppler Kubernetes Operator in your cluster:

```bash
helm install doppler-operator doppler/doppler-operator --namespace doppler-operator-system --create-namespace
```

Verify that the Doppler Operator is installed and running:

```bash
kubectl get pods --namespace doppler-operator-system
```

### 3. Create Doppler Token Secret

To fetch secrets from your Doppler config, you'll need to create a Doppler Token Secret.

#### Option 1: Manually Generate Token and Create Secret

Generate a Doppler Service Token from the Doppler dashboard and create the Kubernetes secret using the following command:

```bash
kubectl create secret generic doppler-token-secret   --namespace doppler-operator-system   --from-literal=serviceToken=dp.st.dev.XXXX
```

Replace `dp.st.dev.XXXX` with your actual Doppler Service Token.

#### Option 2: Generate Token with Doppler CLI

If you have the Doppler CLI installed, you can generate a Service Token and create the secret in one step:

```bash
kubectl create secret generic doppler-token-secret   --namespace doppler-operator-system   --from-literal=serviceToken=$(doppler configure get token --plain)
```

### 4. Create DopplerSecret CRD

Next, create a DopplerSecret CRD that references the Doppler Token Secret and specifies the name and namespace of the managed secret in Kubernetes.

```yaml
apiVersion: secrets.doppler.com/v1alpha1
kind: DopplerSecret
metadata:
  name: dopplersecret-test
  namespace: doppler-operator-system
spec:
  tokenSecret:
    name: doppler-token-secret
  managedSecret:
    name: doppler-test-secret
    namespace: default
    type: Opaque
```

Apply the CRD:

```bash
kubectl apply -f dopplersecret.yaml
```

### 5. Verify Secret Creation

After applying the CRD, verify that the managed secret has been created by the operator:

```bash
kubectl describe secrets --selector=secrets.doppler.com/subtype=dopplerSecret
```

## Using Secrets in Deployments

You can use the managed secret in your Kubernetes deployments in several ways.

### 6. Use `envFrom` to Populate Environment Variables

```yaml
envFrom:
  - secretRef:
      name: doppler-test-secret
```

### 7. Use `valueFrom` to Inject Specific Environment Variables

```yaml
env:
  - name: MY_APP_SECRET
    valueFrom:
      secretKeyRef:
        name: doppler-test-secret
        key: MY_APP_SECRET
```

### 8. Use Secret as a Volume

Create a volume from the secret:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: doppler-test-secret
```

Mount the volume to the container’s filesystem:

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
    readOnly: true
```

## Automatic Redeployments

### 9. Enable Automatic Redeployments

To enable automatic redeployments when secrets are updated:

- Ensure the deployment is in the same namespace as the managed secret.
- Ensure the deployment is of the `Deployment` resource type.
- Add the `secrets.doppler.com/reload` annotation set to 'true'.

Example:

```yaml
annotations:
  secrets.doppler.com/reload: 'true'
```

The operator will update the annotation with the name `secrets.doppler.com/secretsupdate.<KUBERNETES_SECRET_NAME>` to trigger a redeployment.

## Conclusion

By following this guide, you've successfully integrated Doppler with your Kubernetes cluster using Helm, enabling automatic secret management and sync between Doppler and Kubernetes.
