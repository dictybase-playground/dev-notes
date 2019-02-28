* Install minikube

```
$ brew cask install minikube
```

* Verify kubectl is installed

```
$ kubectl version
```

* If not, install that

```
$ brew install kubernetes-cli
```

* Install the [Hyperkit Driver](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver)

```
brew install docker-machine-driver-hyperkit

# docker-machine-driver-hyperkit need root owner and uid 
sudo chown root:wheel /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit
sudo chmod u+s /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit
```

* Start Minikube with RBAC, Hyperkit and specific Kubernetes version

```
$ minikube start --cpus=4 --memory=4096 --disk-size=30g --vm-driver=hyperkit \ 
--kubernetes-version=v1.10.12 --extra-config=apiserver.authorization-mode=RBAC
```

* Install helm

```
$ brew install kubernetes-helm
```

* Create `rbac-config.yaml` file for RBAC config

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

* Run the following and observe its output

```
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created
$ helm init --service-account tiller
```

* Add dictybase helm repository

```
$ helm repo add dictybase https://dictybase-docker.github.io/kubernetes-charts
```

* Follow along with the rest of the [deployment guide](https://github.com/dictyBase/Migration/blob/master/deploy.md).

For Nats, you need to add these three config files.

* account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nats-operator
  namespace: dictybase
```

* rolebinding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dictybase:nats-operator-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dictybase:nats-operator
subjects:
- kind: ServiceAccount
  name: default
  namespace: dictybase
```

* role.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dictybase:nats-operator
rules:
# Allow creating CRDs
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs: ["*"]
# Allow all actions on NatsClusters
- apiGroups:
  - nats.io
  resources:
  - natsclusters
  - natsserviceroles
  verbs: ["*"]
# Allow actions on basic Kubernetes objects
- apiGroups: [""]
  resources:
  - configmaps
  - secrets
  - pods
  - services
  - serviceaccounts
  - serviceaccounts/token
  - endpoints
  - events
  verbs: ["*"]
```

Use kubectl to add each
* `$ kubectl apply -f account.yaml`
* `$ kubectl apply -f rolebinding.yaml`
* `$ kubectl apply -f role.yaml`

Need to add config files for ArangoDB as well.

* adb-account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: arango-deployment-operator
  namespace: dictybase
```

* adb-rolebinding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dictybase:arangodb-deployment-operator-default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dictybase:arangodb-deployment-operator
subjects:
  - kind: ServiceAccount
    name: default
    namespace: dictybase
```

* adb-role.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dictybase:arangodb-deployment-operator
rules:
  - apiGroups: ["database.arangodb.com"]
    resources: ["arangodeployments"]
    verbs: ["*"]
  - apiGroups: ["apiextensions.k8s.io"]
    resources: ["customresourcedefinitions"]
    verbs: ["get"]
  - apiGroups: [""]
    resources:
      [
        "pods",
        "services",
        "endpoints",
        "persistentvolumeclaims",
        "events",
        "secrets",
      ]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["*"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list"]
```

Use kubectl to add each
* `$ kubectl apply -f adb-account.yaml`
* `$ kubectl apply -f adb-rolebinding.yaml`
* `$ kubectl apply -f adb-role.yaml`
