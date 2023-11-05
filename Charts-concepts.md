## Charts
### Dependencies
- WIP
### Template
stores in a chart templates folder, /templates
Values for the templates are supplied into 2 ways:
1. values.yaml inside of a chart, and can contain defaults values
2. Chart users may supply a YAML file that contains values, and can be provided with "helm install"

When a user supplies custom values, these values will OVERRIDE the values in the charts value.yaml file, like values-uat.yaml.

### Predefined values
- On a template, the predefined values are supplied via a values.yaml file or via --set flag. These are accesible from the .Values object in a template.

Others values are avaiable, and cannot be overriden:
- Release.Name
- Release.Namespace
- Release.Service
- Release.IsUpgrade

### Values files
A values file is formatted in YAML. A chart may include a default values.yaml file. The Helm install command allows a user to override values by supplying additional YAML values:
> helm install --generate-name --values=myvals.yaml wordpress

- When values are passed in this way, they will be merged into the default values file

- NOTE: If the --set flag is used on helm install or helm upgrade, those values are simply converted to YAML on the client side.

### Scope, Dependencies, and Values
Values can be on top-level chart and in charts/ dir.
So, values file can supply values to chart as well as to any dependices

```
title: "My WordPress Site" # Sent to the WordPress template

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080

```
So the WordPress chart can access the MySQL password as .Values.mysql.password.
But lower level charts cannot access things in parent charts, so MySQL will not be able to access the title property. Nor, for that matter, can it access apache.port

### Global values
After tittle, you can add a global variable:

```
global:
  app: MyWordPress
```
This value is available to ALL charts as .Values.global.app
And can be added to others charts, lie this:

```
title: "My WordPress Site" # Sent to the WordPress template

global:
  app: MyWordPress

mysql:
  global:
    app: MyWordPress
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  global:
    app: MyWordPress
  port: 8080 # Passed to Apache
```
This provides a way of sharing one top-level variable with all subcharts, which is useful for things like setting metadata properties like labels.
If a subchart declares a global variable, that global will be passed downward (to the subchart's subcharts), but not upward to the parent chart.
**There is no way for a subchart to influence the values of the parent chart.**

### Schema files
A scheme is represented as a JSON schema

### Custom resource definition (CRD)
The CRD are installed before the rest of te chart, and are subject to some limitations.
It CRD yaml files should be in crds/ inside of a chart, many crds can be in the same file.
CRD files cannot be templated in crds/crontab.yaml, they must be plain YAML documents. But, in templates/mycrontab.yaml can be use variables like .Values.name. And, Helm will make sure that Crontab kind will be installed and avaiable form K8s API server BEFORE it proceeds installing the things in templates/

### CRD limits:
- CRD are never reinstalled, Helm wont attempt to install or upgrade.
- CRD are never installed on upgrade or rollback. Helm will only create CRDs on installatin operations.
- CRD are never deleted, if a CRD is deleted, all CRDs contents wiil be deleted across all namespaces in the cluster, Helm wont delete CRDs.


### Using Helm to manage Charts
- Can create a new chart, using this commands:
> helm create mychart
- Helm can package it into a char archive for you:
> helm package mychart
- Helm help you find issues:
> helm lint mychart

### Chart repositories
- A special file called, index.yaml, that has a list of all of the packages supplied by the repository, together with metadata that allows retrieving and verifying those packages.
- From a client side, repos are managed with "helm repo" commands, but Helm dont have any tools for uploading charts to remote repos servers.

### Chart starter packs
#WIP

### Chart Hooks
It allows to INTERVENE at certain points in a release's life cycle, for example:
- Load a ConfigMap or Secret during install before any others charts are loaded.
- Execute a Job to backup a database before installing a new chart, and then execurte a second job after the upgrade in order to restore data.
- Run a job before deleting a release to gracefully take a service out of rotation before removing it.

Hooks are a similiar tool like templates.




