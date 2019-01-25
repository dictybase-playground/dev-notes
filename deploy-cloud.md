Install minikube

```
brew cask install minikube
```

Verify kubectl is installed

```
kubectl version
```

If not, install that

```
brew install kubernetes-cli
```

Install the [Hyperkit Driver](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver)

Start Minikube with RBAC, Hyperkit and specific Kubernetes version

```
minikube start --cpus=4 --memory=4096 --disk-size=30g --vm-driver=hyperkit --kubernetes-version=v1.10.12 --bootstrapper=kubeadm --extra-config=apiserver.authorization-mode=RBAC
```

Install helm

```
brew install kubernetes-helm
```

Create `rbac-config.yaml` file for RBAC config

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Run the following and observe its output

```
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created
$ helm init --service-account tiller
```

Add dictybase helm repository

```
helm repo add dictybase https://dictybase-docker.github.io/kubernetes-charts
```

Follow along with the rest of the [deployment guide](https://github.com/dictyBase/Migration/blob/master/deploy.md).
