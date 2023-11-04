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
The first one is for charts from remote repositories, and the second one for your locals charts (added using helm repo add)
