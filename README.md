# kubevirt-helm

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

## Flux / GitOps

These charts have no values, so Flux deploys them as plain HelmReleases. Point one
`HelmRepository` at the chart repo, and a `HelmRelease` per chart. Because the
chart CRs ship with empty/short `featureGates`, patch them back via
`postRenderers` (Flux's `kustomize` support) to what the cluster actually runs.

### HelmRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: kubevirt
  namespace: kube-system
spec:
  url: https://general-intelligence-systems.github.io/kubevirt-helm
  interval: 1h
```

### KubeVirt operator

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: kubevirt-operator
  namespace: kube-system
spec:
  interval: 30m
  targetNamespace: kubevirt
  chart:
    spec:
      chart: kubevirt-operator
      version: "1.9.0"
      sourceRef:
        kind: HelmRepository
        name: kubevirt
        namespace: kube-system
  install:
    crds: Create
    remediation:
      remediateLastFailure: false
  upgrade:
    crds: CreateReplace
    remediation:
      remediateLastFailure: false
  driftDetection:
    mode: enabled
  postRenderers:
  - kustomize:
      patches:
      - target:
          kind: Namespace
          name: kubevirt
        patch: |
          apiVersion: v1
          kind: Namespace
          metadata:
            name: kubevirt
            annotations:
              helm.sh/resource-policy: keep
```

> The `helm.sh/resource-policy: keep` annotation stops Helm deleting the
> namespace if a future uninstall ever runs, since the chart ships its own
> `kind: Namespace`.

### KubeVirt custom resource

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: kubevirt-cr
  namespace: kube-system
spec:
  interval: 30m
  targetNamespace: kubevirt
  dependsOn:
  - name: kubevirt-operator
  chart:
    spec:
      chart: kubevirt-cr
      version: "1.9.0"
      sourceRef:
        kind: HelmRepository
        name: kubevirt
        namespace: kube-system
  install:
    remediation:
      remediateLastFailure: false
  upgrade:
    remediation:
      remediateLastFailure: false
  driftDetection:
    mode: warn
  postRenderers:
  - kustomize:
      patches:
      - target:
          kind: KubeVirt
          name: kubevirt
        patch: |
          - op: replace
            path: /spec/configuration/developerConfiguration/featureGates
            value:
            - Sidecar
            - DeclarativeHotplugVolumes
```

### CDI operator

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: cdi-operator
  namespace: kube-system
spec:
  interval: 30m
  targetNamespace: cdi
  chart:
    spec:
      chart: cdi-operator
      version: "1.65.0"
      sourceRef:
        kind: HelmRepository
        name: kubevirt
        namespace: kube-system
  install:
    crds: Create
    remediation:
      remediateLastFailure: false
  upgrade:
    crds: CreateReplace
    remediation:
      remediateLastFailure: false
  driftDetection:
    mode: enabled
  postRenderers:
  - kustomize:
      patches:
      - target:
          kind: Namespace
          name: cdi
        patch: |
          apiVersion: v1
          kind: Namespace
          metadata:
            name: cdi
            annotations:
              helm.sh/resource-policy: keep
```

### CDI custom resource

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: cdi-cr
  namespace: kube-system
spec:
  interval: 30m
  targetNamespace: cdi
  dependsOn:
  - name: cdi-operator
  chart:
    spec:
      chart: cdi-cr
      version: "1.65.0"
      sourceRef:
        kind: HelmRepository
        name: kubevirt
        namespace: kube-system
  install:
    remediation:
      remediateLastFailure: false
  upgrade:
    remediation:
      remediateLastFailure: false
  driftDetection:
    mode: warn
  postRenderers:
  - kustomize:
      patches:
      - target:
          kind: CDI
          name: cdi
        patch: |
          - op: replace
            path: /spec/config/featureGates
            value:
            - HonorWaitForFirstConsumer
            - WebhookPvcRendering
```

> On the operator CRs, `remediateLastFailure: false` stops helm-controller
> uninstalling the release (and, via the charts' own `Namespace` resources, the
> whole namespace) if the first install fails. `driftDetection: warn` (rather
> than `enabled`) on the CR releases is because the operators default absent
> spec fields on the CR they own, which `enabled` would read back as drift and
> re-apply forever.
