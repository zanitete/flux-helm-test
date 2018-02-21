# flux-helm-test
A repository with test data for flux-helm integration

# Helm Integration - alpha

## Synopsis

Helm integration provides an extension to Flux (https://github.com/weaveworks/flux) to be able to deal with Helm Chart releases.
A Chart release is described through a custom resource of a bespoke kubernetes kind. A custom resource contains all that needs to be known to do a Chart release (see Custom resource section). At this stage, Helm charts and chart release configuration need to share one repo.

Helm integration has three parts:

| helm integration part                   | purpose                       |
|------------------------|-------------------------------|
| CustomResourceDefinition for bespoke custom resources | Custom resources (CR) provide chart release configuration/desired state |
| Flux agent                                            | Monitors chart release configurations. On finding git changes applies CR manifests |
| Helm operator                                         | Watches for kubernetes events related to custom resources of FluxHelmRelease kind and acts accordingly on their creation/update/deletion by creating/updating/deleting the relevant release. Also monitors the repo’s charts path and updates the relevant release(s)|

## Custom resource

### Example
```
---
  apiVersion: helm.integrations.flux.weave.works/v1alpha
  kind: FluxHelmRelease
  metadata:
    name: mongodb
    namespace:  myNamespace
    labels:
      chart: mongodb
  spec:
    chartGitPath: mongodb
    releaseName: mongo-database
    values:
      - name: image
        value: bitnami/mongodb:3.7.1-r1
      - name: imagePullPolicy
        value: IfNotPresent
```

 - custom resource (CR) name is arbitrary (needs to follow k8s naming conventions):
 - namespace is optional (default by default). Determines where the CR and the Chart are deployed:
  - If releaseName is not provided, the release name will be created as $namespace-$customResourceName
 - labels.chart corresponds to chartGitPath, ie chart name
 - chartGitPath is the chart subdir under the charts path (provided to helm-operator together with the rest of the ). Equivalent to the chart name
 - releaseName: optional. Needs to be provided if there is already a running chart release in the cluster and the user wants Flux to manage it from now on.
 - values are user customizations of default parameter values

## Prerequisites

- *Helm server tiller should be ideally running in the cluster already, though the helm-operator will wait until it does.* 
- *Git repo with the following (required) structure:*

```
gitUser/repoName/charts/chart1# 
gitUser/repoName/charts/chart2 etc

gitUser/repoName/releaseconfig/customResource1.yaml
gitUser/repoName/releaseconfig/customResource2.yaml etc
```

### Note

1. Path with chart release configuration (gitUser/repoName/releaseconfig) is provided in the Weave Cloud GUI

2. Path containing charts (as subdirectories directly under this path) is provided to helm-operator in its deployment manifest

3. Name of either path is configurable:
  - releaseconfig path (containing custom resources manifests) is provided through the Weave Cloud UI (used by flux agent)
  - charts path can be configured in the helm-operator-deployment.yaml (used by helm-operator)
	
4. One releaseconfig yaml file can contain multiple custom resource manifests

## User setup

1. A cluster available
2. Set up helm by running: `helm init`
3. Make a fork of https://github.com/weaveworks/flux-helm-test
4. Change the git repo information in the artifacts/weave-helm-operator.yaml to refer to the forked one
5. Go to Weave Cloud and create an instance
6. Set up Weave agents by running the provided curl command
7. Before providing git repo information in the GUI, run the following command locally (from the forked repo root): `kubectl apply -f artifacts/weave-helm-operator.yaml`
8. Provide git repo information in the Weave Cloud UI
9. All should be set up and running
