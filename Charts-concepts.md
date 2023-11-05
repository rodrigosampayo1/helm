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
- pre-install:
Executes after templates are rendered, but before any resources are created in K8s.
- post-install:
Executes after all resources are loaded into K8s.
- pre-delete:
Executes on a deletion request before any resources are deleted from K8s.
- post-delete:
Executes on a deletion request after all of the release's resources have been deleted.
- pre-upgraded
- Post-upgraded
- pre-rollback
- post-rollback
- test

Hooks allows you an opportuniyty to perform operations at strategic points in a release lifecycle.
For example, consider the lifecycle for a helm install
1. Users runs helm install foo
2. Helm library install API is called
3. The library rendes the foo templates
4. The library loads the resulting resources into K8s
5. The library returns the release oboject to the client
6. The client exits.
As Helms defines 2 hooks for install, pre and post-install, you can change the lifecycle like this:

1. Users runs helm install foo
2. Helm library install API is called
3. CRDs in the crds/ dir are installed.
4. The library rendes the foo templates
5. The library prepares to execute the pre-install hooks.
6. The library sorts hooks by weight, by resource kind and finally by name.
7. The library then loads the hook wit the lowest weight first.
8. The library waits unitl the hook is ready, except for CRDs
9. The library loads the resulting resources into K8s.
10. THe library executes the post-install hook, loading hook resources.
11. The library waits unitl the hooks is ready.
12. The library retunrs the release object to the client
13. The client exits.


### Writing a Hook
Hooks are just Kubernetes manifest files with special annotations in the metadata section. SIMILAR to template but not same.
Because they are template files, you can use all of the normal template features, including reading .Values, .Release, and .Template.

What makes this template a hook is the annotation:
> annotations:
>   "helm.sh/hook": post-install

Hook weights can be positive or negative numbers but must be represented as strings. When Helm starts the execution cycle of hooks of a particular Kind it will sort those hooks in ascending order.

### Chart tests
- As a chart author, you may want to write some tests that validate that your chart works as expected when it is installed. 
- These tests also help the chart consumer understand what your chart is supposed to do.
- A test in a helm chart lives under the templates/ directory and is a job definition that specifies a container with a given command to run.
- The job definition must contain the helm test hook annotation: helm.sh/hook: test.

Example tests:

- Validate that your configuration from the values.yaml file was properly injected.
- Make sure your username and password work correctly
- Make sure an incorrect username and password does not work
- Assert that your services are up and correctly load balancing.

You can run the pre-defined tests in Helm on a release using the command:
> helm test <RELEASE_NAME>

- Example:
1. helm create demo
(it creates a basic release, with Chart.yaml, values.yaml, charts/, templates/)
- In demo/templates/tests/test-connection.yaml you'll see a test you can try.

### Steps to Run a Test Suite on a Release
1. Install the chart
> helm install demo demo --namespace default

2. helm test demo

- A test is a Helm hook, so annotations like helm.sh/hook-weight and helm.sh/hook-delete-policy may be used with test resources.

