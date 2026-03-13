# kubevirt-helm

Helm charts for [KubeVirt](https://kubevirt.io), automatically generated from the upstream [kubevirt/kubevirt](https://github.com/kubevirt/kubevirt) GitHub releases. All releases from v0.12.0 onwards are included.

## Install

Add the repo:

```bash
helm repo add kubevirt https://general-intelligence-systems.github.io/kubevirt-helm
helm repo update
```

Install the operator and CR (latest version):

```bash
helm install kubevirt-operator kubevirt/kubevirt-operator
helm install kubevirt-cr kubevirt/kubevirt-cr
```

Install a specific version:

```bash
helm install kubevirt-operator kubevirt/kubevirt-operator --version 1.7.0
helm install kubevirt-cr kubevirt/kubevirt-cr --version 1.7.0
```

## Upgrade

```bash
helm repo update
helm upgrade kubevirt-operator kubevirt/kubevirt-operator
helm upgrade kubevirt-cr kubevirt/kubevirt-cr
```

## Uninstall

```bash
helm uninstall kubevirt-cr
helm uninstall kubevirt-operator
```

## Available charts

| Chart | Description |
|---|---|
| `kubevirt-operator` | KubeVirt operator deployment and CRDs |
| `kubevirt-cr` | KubeVirt custom resource (triggers the operator to deploy KubeVirt) |

## How it works

These charts are the Helm-packaged equivalent of the [KubeVirt quickstart](https://kubevirt.io/quickstart_cloud/):

```bash
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml"
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml"
```

Each chart contains the upstream YAML manifests as-is. There are no Helm values or templating — the charts exist purely to provide Helm lifecycle management (install, upgrade, rollback, uninstall) for KubeVirt.
