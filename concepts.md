# Helm Concepts
## Chart
A char is a packaging format, a collection of files that describe a related set of Kubernetes resources.
It's a helm package, like brew, apt, yum. It helps you to install app in your cluster.
- To see withoud download, you can use
> helm pull chartrepo/chartname

### Chart types:
1. Application (the default type)
2. Library (provides utilities or functions for the chart builder. It is not installable and usually does not contain any resource objects).

### Chart dependencies
Any dependencies can be dynamically linked using the dependencies field in Chart.yaml or brought in to the charts/ directory and managed manually.
```
dependencies:
  - name: apache
    version: 1.2.3
    repository: https://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: https://another.example.com/charts

```
 helm repo add fantastic-charts https://charts.helm.sh/incubator

```
dependencies:
  - name: awesomeness
    version: 1.0.0
    repository: "@fantastic-charts"
```

To update a dependency, you can run this:
> helm dep up foochart

## Repository
It's a hub where charts can be founded and shared among us for Kubernetes packages.

## Release
It's a instance of a chart running in a cluster
Like an imagen and container. Every time that a chart is installed, a release is deployed. 1 Charts can have N releases.

## Find a chart, 2 options local or remotly
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
- helm upgrade, is for a new version of a char is realisded or when you want to change the config of your release

```
helm upgrade -f panda.yaml happy-panda bitnami/wordpress
```

Helm tries to perform the least invasive upgrade, only update things that have change since the last release.
e.g.:
> helm upgrade [RELEASE] [CHART] [flags]

### CHART can be:
- a chart reference
- a path to a chart directory
- a package chart
- full qualified URL

- Can add a flag, --version or the lastest for default.
- Can use a flag for --values, to overrid a values in a chart
- Can use a flag --set to pass config from th command line
- Can use --set-string, to force string values
- Can use --set-file to set individual values from a file when the value itself is too long for the command line or is dynamically generated.
- Can use --set-json to set json values (scalars, objects, arrays) from the CLI
- Can use --values'/'-f falg multiple times. Had more precedens the right-most file.
example:
helm upgrade -f myvalues.yaml -f override.yaml reids ./redis
helm upgrade --set foo=bar --set foo=newbar redis ./redis

- Can use --reuse-values flag, to update the values for an existing release
example: 
helm upgrade --reuse-values --set foo=bar --set foo=newbar redis ./redis


## Helm verify
verify that a chart at the given path has been signed and is valid

> helm verify PATH [flags]

## Helm Version
print the client version information
> helm version [flags]

