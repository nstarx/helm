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
| operator.watchNamespace | string | `"default"` | Namespace to watch for Kubeconfig resources |
| operator.logLevel | string | `"info"` | Log level (debug, info, warning, error) |
| operator.notifications.enabled | bool | `true` | Enable notifications |
| operator.notifications.endpoint | string | `"http://notification-service:8080/send-message"` | Notification endpoint URL |

## Usage

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