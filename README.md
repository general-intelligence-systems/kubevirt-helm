# kubevirt-helm

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/general-intelligence-systems/kubevirt-helm)

Helm charts for [KubeVirt](https://kubevirt.io) and [CDI](https://github.com/kubevirt/containerized-data-importer), automatically generated from upstream GitHub releases. All releases from v0.12.0 onwards are included for KubeVirt, and all CDI releases are included.

## Install

Add the repo:

```bash
helm repo add kubevirt https://general-intelligence-systems.github.io/kubevirt-helm
helm repo update
```

Install KubeVirt (latest version):

```bash
helm install kubevirt-operator kubevirt/kubevirt-operator
helm install kubevirt-cr kubevirt/kubevirt-cr
```

Install CDI (latest version):

```bash
helm install cdi-operator kubevirt/cdi-operator
helm install cdi-cr kubevirt/cdi-cr
```

Install specific versions:

```bash
helm install kubevirt-operator kubevirt/kubevirt-operator --version 1.7.0
helm install kubevirt-cr kubevirt/kubevirt-cr --version 1.7.0
helm install cdi-operator kubevirt/cdi-operator --version 1.65.0
helm install cdi-cr kubevirt/cdi-cr --version 1.65.0
```

## Upgrade

```bash
helm repo update
helm upgrade kubevirt-operator kubevirt/kubevirt-operator
helm upgrade kubevirt-cr kubevirt/kubevirt-cr
helm upgrade cdi-operator kubevirt/cdi-operator
helm upgrade cdi-cr kubevirt/cdi-cr
```

## Uninstall

```bash
helm uninstall kubevirt-cr
helm uninstall kubevirt-operator
helm uninstall cdi-cr
helm uninstall cdi-operator
```

## Available charts

| Chart | Source | Description |
|---|---|---|
| `kubevirt-operator` | [kubevirt/kubevirt](https://github.com/kubevirt/kubevirt) | KubeVirt operator deployment and CRDs |
| `kubevirt-cr` | [kubevirt/kubevirt](https://github.com/kubevirt/kubevirt) | KubeVirt custom resource (triggers the operator to deploy KubeVirt) |
| `cdi-operator` | [kubevirt/containerized-data-importer](https://github.com/kubevirt/containerized-data-importer) | CDI operator deployment and CRDs |
| `cdi-cr` | [kubevirt/containerized-data-importer](https://github.com/kubevirt/containerized-data-importer) | CDI custom resource (triggers the operator to deploy CDI) |

## Implementation

The charts are the equivalent of applying the upstream YAML manifests directly. There are no helm values to apply.

KubeVirt (from the [quickstart guide](https://kubevirt.io/quickstart_cloud/)):

```bash
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml"
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml"
```

CDI (from the [CDI docs](https://github.com/kubevirt/containerized-data-importer#deploy-it)):

```bash
export CDI_VERSION=$(curl -s https://api.github.com/repos/kubevirt/containerized-data-importer/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
kubectl create -f "https://github.com/kubevirt/containerized-data-importer/releases/download/${CDI_VERSION}/cdi-operator.yaml"
kubectl create -f "https://github.com/kubevirt/containerized-data-importer/releases/download/${CDI_VERSION}/cdi-cr.yaml"
```

Each chart contains the upstream YAML manifests as-is. There are no Helm values or templating — the charts exist purely to provide Helm lifecycle management (install, upgrade, rollback, uninstall) for KubeVirt and CDI.
