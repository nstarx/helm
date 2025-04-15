# dlnp-operator

This Helm chart deploys the DLNP Operator, which manages the deployment of Kubeflow to Kubernetes clusters through Kubeconfig custom resources.

## Overview

The dlnp-operator watches for `Kubeconfig` custom resources in a Kubernetes cluster. When a new Kubeconfig resource is created, the operator:

1. Extracts the kubeconfig content from the resource
2. Creates a temporary kubeconfig file
3. Validates connectivity to the referenced cluster
4. Deploys the nstarx-kubeflow-core Helm chart to that cluster
5. Sends a notification upon completion

This enables automated deployment of Kubeflow to multiple clusters by simply creating Kubeconfig resources.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- kubectl configured for your cluster

## Installation

```bash
# Add the repo
helm repo add dlnp https://karrad.github.io/helm

# Update repositories
helm repo update

# Install the operator
helm install dlnp-operator dlnp/dlnp-operator
```

Or install from local chart:

```bash
# Clone the repository
git clone https://github.com/karrad/dlnp-operator.git
cd dlnp-operator

# Install the chart
helm install dlnp-operator ./chart
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| replicaCount | int | 1 | Number of operator replicas |
| image.repository | string | `"karrad/dlnp-operator"` | Image repository |
| image.tag | string | `"latest"` | Image tag |
| image.pullPolicy | string | `"Always"` | Image pull policy |
| operator.watch.namespace | string | `"default"` | Namespace to watch for Kubeconfig resources ("" for all namespaces, "deployed" for same namespace as operator) |
| operator.watch.clusterWide | bool | `false` | Whether to use cluster-wide permissions |
| operator.logLevel | string | `"info"` | Log level (debug, info, warning, error) |
| operator.notifications.enabled | bool | `true` | Enable notifications |
| operator.notifications.endpoint | string | `"http://notification-service:8080/send-message"` | Notification endpoint URL |
| operator.env | list | `[]` | List of environment variables to set in the container |
| operator.envFrom | list | `[]` | List of sources to populate environment variables in the container |
| operator.configMapEnv | list | `[]` | List of ConfigMaps to use as environment variable sources |

## Usage

### Setting Custom Environment Variables

You can set environment variables for the operator container in several ways:

1. Directly in your `values.yaml`:

```yaml
operator:
  env:
    - name: CHART_REPO_URL
      value: "https://my-helm-repo.example.com"
    - name: DEBUG_MODE
      value: "true"
```

2. Using Kubernetes Secrets:

```yaml
operator:
  envFrom:
    - secretRef:
        name: operator-credentials
```

3. Using ConfigMaps:

```yaml
operator:
  configMapEnv:
    - configMapRef:
        name: operator-configuration
```

4. Using Helm's `--set` flag:

```bash
helm install dlnp-operator ./dlnp-operator \
  --set operator.env[0].name=CHART_VERSION \
  --set operator.env[0].value=1.0.0
```

## Namespace Configuration

### Deployment Namespace 

The namespace where the operator itself is deployed is controlled by the `--namespace` flag during Helm installation:

```bash
# Deploy to a specific namespace (creates it if it doesn't exist)
helm install dlnp-operator ./dlnp-operator --namespace operator-system --create-namespace
```

If you don't specify a namespace, Helm will use your current kubectl context's namespace.

### Watch Namespace

The namespace where the operator looks for Kubeconfig resources is configured in the values.yaml file:

```yaml
operator:
  watch:
    # Watch a specific namespace
    namespace: "application-namespace" 
    
    # OR watch the same namespace where the operator is deployed
    # namespace: "deployed"
    
    # OR watch all namespaces 
    # namespace: ""
    # clusterWide: true
```

This gives you complete flexibility to:

1. Deploy the operator in a system/admin namespace
2. Have it watch for resources in application namespaces
3. Or watch the entire cluster if needed

## Watch Configuration Examples

### Watch the same namespace the operator is deployed in:

```yaml
operator:
  watch:
    namespace: "deployed"
    clusterWide: false
```

### Watch a specific different namespace:

```yaml
operator:
  watch:
    namespace: "target-namespace"
    clusterWide: false
```

### Watch all namespaces (cluster-wide):

```yaml
operator:
  watch:
    namespace: ""
    clusterWide: true
```

1. Directly in your `values.yaml`:

```yaml
operator:
  env:
    - name: CHART_REPO_URL
      value: "https://my-helm-repo.example.com"
    - name: DEBUG_MODE
      value: "true"
```

2. Using Kubernetes Secrets:

```yaml
operator:
  envFrom:
    - secretRef:
        name: operator-credentials
```

3. Using ConfigMaps:

```yaml
operator:
  configMapEnv:
    - configMapRef:
        name: operator-configuration
```

4. Using Helm's `--set` flag:

```bash
helm install dlnp-operator ./dlnp-operator \
  --set operator.env[0].name=CHART_VERSION \
  --set operator.env[0].value=1.0.0
```

### Creating a Kubeconfig Resource

To deploy Kubeflow to a target cluster, create a Kubeconfig custom resource:

```yaml
apiVersion: nstarx.com/v1alpha1
kind: Kubeconfig
metadata:
  name: cluster-1
spec:
  cloud: aws
  kubeconfig: |
    apiVersion: v1
    kind: