# kubevirt-helm

This helm repo is automatically generated from the github releases page of the kubevirt git repo.

## Implementation

Basically the charts are the equivalent of doing the following (taken from the [quickstart guide](https://kubevirt.io/quickstart_cloud/)). There are no helm values to apply.

```bash
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)

kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml"
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml"
```


