## Google Cloud Deployment Guide

First create a cluster in [GKE](https://console.cloud.google.com/kubernetes).

- v1.11.8-gke.6
- One node
- Four vCPUs
- 4.00 GB memory
- upgrades off
- SSD boot disk (50 gb)
- disable authorization
- either use given serviceaccount or full cloud access

Get credentials:

>`$_> gcloud container clusters get-credentials <CLUSTER_NAME> --project <PROJECT_NAME>`

- You may also need to pass in `--zone` with the cluster location (i.e. `us-central1-a`)

Verify that you are now connected and using correct context.

>`$_> kubectl config get-contexts`.

### Helm

- Add RBAC for Helm

```shell
$_> kubectl -n kube-system create serviceaccount tiller
$_> kubectl create clusterrolebinding tiller \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
```

- Initialize Helm

>`$_> helm init --service-account=tiller`


### Cluster Role Bindings

- Create a cluster admin role for the current executing user   
First get the email of the current user
`gcloud info | grep Account`

```shell
$_> kubectl create clusterrolebinding dictyadmin \
     --clusterrole=cluster-admin \
     --user=youremail@email.org
```

- Add cluster role binding for nats

```shell
$_> kubectl create clusterrolebinding default \
    --clusterrole=cluster-admin \
    --serviceaccount=dictybase:default
```

### Recommendations

- Install [k9s](https://github.com/derailed/k9s) for terminal dashboard. Run this with the `k9s` command.

### ArangoDB

- Install [kube-arangodb](https://github.com/arangodb/kube-arangodb/blob/0.3.8/docs/Manual/Deployment/Kubernetes/Helm.md)

```shell
$_> helm install https://github.com/arangodb/kube-arangodb/releases/download/0.3.8/kube-arangodb-crd.tgz
$_> helm install https://github.com/arangodb/kube-arangodb/releases/download/0.3.8/kube-arangodb.tgz \
    --set=DeploymentReplication.Create=false --namespace dictybase
```

- Install our database

>`$_>  helm install dictybase/arangodb --namespace dictybase`

#### Create necessary databases

- Make yaml config file (new-db.yaml)

```yaml
database:
  names:
    - order
    - stock
  user: george
  password: costanza
  grant: rw
```

>`$_> helm install dictybase/arango-create-database --namespace dictybase -f new-db.yaml`

#### Add backends

- make sure to use the previously created user (i.e. the one you set while creating the database)

```shell
$_> helm install dictybase/stock-api-server --namespace dictybase \
    --set database.name=stock --set database.user=george \
    --set database.password=costanza
$_> helm install dictybase/order-api-server --namespace dictybase \
    --set database.name=order --set database.user=george \
    --set database.password=costanza
```

### PostgreSQL

- Quick deploy

```shell
$_> helm install dictybase/dictycontent-postgres --namespace dictybase \
		--set postgresPassword=somepass  \
		--set dictycontentPassword=someotherpass \ 
		--set dictyuserPassword=someotherpass \ 
```

#### Schema loaders

```shell
$_> helm install dictybase/dictycontent-schema --namespace dictybase
$_> helm install dictybase/dictyuser-schema --namespace dictybase
```

### API Services

#### `Content`
> `$_> helm install dictybase/content-api-server --namespace dictybase \`   
>		`--set apiHost=https://betaapi.dictybase.local`

#### `User`
> $_> `helm install dictybase/user-api-server --namespace dictybase \`   
>		`--set apiHost=https://betaapi.dictybase.local`

#### `Order`
```shell
$_> helm install dictybase/order-api-server --namespace dictybase \
    --set database.name=order --set database.user=george \
    --set database.password=costanza
```

#### `Stock`
```shell
$_> helm install dictybase/stock-api-server --namespace dictybase \
    --set database.name=stock --set database.user=george \
    --set database.password=costanza
```


### GraphQL

- Add GraphQL server

>`$_> helm install dictybase/graphql-server --namespace dictybase`
