# helm
## Prerequisites
- A K8s cluster
- Install and config helm
helm list

## Installation Helm guide
https://helm.sh/docs/intro/install/


## Helm commands
- Add Helm repo
```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm search repo bitnami
```

- Install a chart
```
helm repo update
helm install bitnami/mysql --generate-name [-n default]
```

- See releases
```
helm list
#or
helm ls
```

- Uninstall a release
```
helm uninstall release-name
helm unistall release-name --keep-history
```

- Helm help
```
helm get -h
```

## User guide
https://helm.sh/docs/intro/using_helm/
