# Helm Concepts
## Chart
It's a helm package, like brew, apt, yum. It helps you to install app in your cluster.

## Repository
It's a hub where charts can be founded and shared among us for Kubernetes packages.

## Release
It's a instance of a chart running in a cluster
Like an imagen and container. Every time that a chart is installed, a release is deployed. 1 Charts can have N releases.

## Find a chart
```
helm search hub

#or
Helm search repo
```
- The first one is for charts from remote repositories, and the second one for your locals charts (added using helm repo add)


```
helm status release-name

```
```
helm show values chart-name

```
- You can override any of these settings in a YAML formated, using values.yaml, and use multiple times
And then use it:
```
helm install -f values.yaml bitnami/wordpress --generate-name
```

The helm install can install from several sources:
- A chart repo
- a local chart archive
- Unpacked chart directory
- A full URL

## Upgrading a release and recovering on failure
```
helm upgrade -f panda.yaml happy-panda bitnami/wordpress
```