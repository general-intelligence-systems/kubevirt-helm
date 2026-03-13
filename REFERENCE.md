KubeVirt.io
Blogs
Videos
Gallery
Docs
Labs
Community
search term
 
Labs - KubeVirt quickstart with Minikube
Quickstart Guides
Minikube
Kind
Cloud providers
Labs
Use KubeVirt
Experiment with CDI
KubeVirt Upgrades
Live Migration
Easy install using minikube
Minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows allowing software developers to quickly get started working with Kubernetes.

Prepare minikube Kubernetes environment
A kubectl client is necessary for operating a Kubernetes cluster. It is important to install a kubectl client version that matches the kubernetes version to avoid issues regarding skew.

To install kubectl client please follow the official documentation for your system using the instructions located here.
Minikube ships a kubectl client version that matches the kubernetes version to avoid skew issues. To use the minikube shipped client do one of the following:

All normal kubectl commands should be performed as minikube kubectl
It can be added to aliases by running the following:

Copy
alias kubectl='minikube kubectl --'
It can be installed directly to the host by running the following:

Copy
VERSION=$(minikube kubectl version | head -1 | awk -F', ' {'print $3'} | awk -F':' {'print $2'} | sed s/\"//g)
sudo install -m 0755 ${HOME}/.minikube/cache/linux/${VERSION}/kubectl /usr/local/bin
To install minikube please follow the official documentation for your system using the instructions located here.

Starting minikube can be as simple as running the following command:

Copy
minikube start --cni=flannel
CNI:

We add the container network interface (CNI) called flannel to make minikube work with VMs that use a masquerade type network interface. If a CNI does not work for you, switch instances of “masquerade” to “bridge” in example VM definitions.

See the minikube handbook here for advanced start options and instructions on how to operate minikube.

Multi-Node Minikube
Minikube has support for adding additional nodes to a cluster. This can be helpful in experimenting with KubeVirt on minikube as some operations like node affinity or live migration require more than one cluster node to demonstrate.

Container Network Interface
By default, minikube sets up a kubernetes cluster using either a virtual machine appliance or a container. For a single node setup, local network connectivity is sufficient. In the case where multiple nodes are involved, even when using containers or VMs on the same host, kubernetes needs to define a shared network to allow pods on one host to communicate with pods on the other host. To this end, minikube supports a number of Container Network Interface (CNI) plugins the simplest of which is flannel.

Updating the minikube start command
To have minikube start up with the flannel CNI plugin over two nodes, alter the minikube start command:

Copy
minikube start --nodes=2 --cni=flannel
Core DNS race condition

An issue has been reported where the coredns pod in multi-node minikube comes up with the wrong IP address. If this happens, kubevirt will fail to install properly. To work around, delete the coredns pod from the kube-system namespace and disable/enable the kubevirt addon in minikube.

Deploy KubeVirt
KubeVirt can be installed using the KubeVirt operator, which manages the lifecycle of all the KubeVirt core components.

Below are two examples of how to install KubeVirt using the latest release.

The easy way
Addon currently broken

An issue has been reported where more recent versions of minikube break the kubevirt addon. Fall back to the “in-depth” section below until this is resolved.

Installing KubeVirt can be as simple as the following command:

Copy
minikube addons enable kubevirt
The in-depth way
Use kubectl to deploy the KubeVirt operator:

Copy
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
echo $VERSION
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml"
Nested virtualization

If the minikube cluster runs on a virtual machine consider enabling nested virtualization. Follow the instructions described here. If for any reason nested virtualization cannot be enabled do enable KubeVirt emulation as follows:

Copy
kubectl -n kubevirt patch kubevirt kubevirt --type=merge --patch '{"spec":{"configuration":{"developerConfiguration":{"useEmulation":true}}}}'
Again use kubectl to deploy the KubeVirt custom resource definitions:

Copy
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml"
Verify components
By default KubeVirt will deploy 7 pods, 3 services, 1 daemonset, 3 deployment apps, 3 replica sets.

Check the deployment:

Copy
kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
Check the components:

Copy
kubectl get all -n kubevirt
When using the minikube KubeVirt addon check logs of the kubevirt-install-manager pod:

Copy
kubectl logs pod/kubevirt-install-manager -n kube-system
Virtctl
KubeVirt provides an additional binary called virtctl for quick access to the serial and graphical ports of a VM and also handle start/stop operations.

Install
virtctl can be retrieved from the release page of the KubeVirt github page.

Run the following:
Copy
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
sudo install -m 0755 virtctl /usr/local/bin
Install as Krew plugin
virtctl can be installed as a plugin via the krew plugin manager. Occurrences of virtctl <command>... can then be read as kubectl virt <command>....

Run the following to install:
Copy
kubectl krew install virt


What’s next: Labs
After you have deployed KubeVirt you can work through the labs to help you get acquainted with KubeVirt and how it can be used to create and deploy VMs with Kubernetes.

The first lab is “Use KubeVirt”. This lab walks through the creation of a Virtual Machine Instance (VMI) on Kubernetes and then how virtctl is used to interact with its console.

The second lab is “Experiment with CDI”. This lab shows how to use the Containerized Data Importer (CDI) to import a VM image into a Persistent Volume Claim (PVC) and then how to attach the PVC to a VM as a block device.

The third lab is “KubeVirt upgrades”. This lab shows how easy and safe is to upgrade the KubeVirt installation with zero down-time.

Found a bug?
We are interested in hearing about your experience.

Please report any problems to the kubevirt.io issue tracker.

We are a Cloud Native Computing Foundation incubating project.

Cloud Native Computing Foundation

      

Copyright 2026 The KubeVirt Contributors
Copyright 2026 The Linux Foundation. All Rights Reserved. The Linux Foundation has registered trademarks and uses trademarks. For a list of trademarks of The Linux Foundation, please see our Trademark Usage page.
This site is powered by Netlify.
Privacy Statement
