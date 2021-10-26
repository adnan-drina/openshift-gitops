# openshift-gitops

Red Hat OpenShift GitOps is a declarative continuous delivery platform based on [Argo CD](https://argoproj.github.io/argo-cd/). It enables teams to adopt GitOps principles for managing cluster configurations and automating secure and repeatable application delivery across hybrid multi-cluster Kubernetes environments. Following GitOps and infrastructure as code principles, you can store the configuration of clusters and applications in Git repositories and use Git workflows to roll them out to the target clusters.


This repository explains how to install the Red Hat OpenShift GitOps Operator to an OpenShift Container Platform cluster and logging in to the Argo CD instance.

**Prerequisites**
- Access to the OpenShift Container Platform web console.
- An account with the cluster-admin role.
- You are logged in to the OpenShift Container Platform cluster as an administrator.

Documentation [Getting started with OpenShift GitOps](https://docs.openshift.com/container-platform/4.8/cicd/gitops/installing-openshift-gitops.html)

## Installing GitOps Operator using CLI

Ensure that the pipeline operator exists in the channel catalog.
```shell script
oc get packagemanifests -n openshift-marketplace | grep openshift-gitops
```

Query the available channels for gitops operator
```shell script
oc get packagemanifest -o jsonpath='{range .status.channels[*]}{.name}{"\n"}{end}{"\n"}' -n openshift-marketplace openshift-gitops-operator
```

Discover whether the operator can be installed cluster-wide or in a single namespace
```shell script
oc get packagemanifest -o jsonpath='{range .status.channels[*]}{.name}{" => cluster-wide: "}{.currentCSVDesc.installModes[?(@.type=="AllNamespaces")].supported}{"\n"}{end}{"\n"}' -n openshift-marketplace openshift-gitops-operator
```

Check the operator information for additional details
```shell script
oc describe packagemanifests/openshift-gitops-operator -n openshift-marketplace
```

Create a Subscription object YAML file to subscribe a namespace to the Red Hat OpenShift GitOps Operator:

**Example Subscription**

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-operators
spec:
  channel:  <channel name> #Specify the channel name from where you want to subscribe the Operator
  name: openshift-pipelines-operator-rh #Name of the Operator to subscribe to.
  source: redhat-operators #Name of the CatalogSource that provides the Operator.
  sourceNamespace: openshift-marketplace #Namespace of the CatalogSource. Use openshift-marketplace for the default OperatorHub CatalogSources.
```

-   Create the Subscription object:

```bash
$ oc apply -f gitops-sub.yaml
```
The Red Hat OpenShift Gitops Operator is now installed in the default target namespace openshift-operators.

Now we you can install Argo CD which is the representation of a GitOps deployment.

***Example ArgoCD***

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: openshift-operators
spec:
  server:
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 125m
        memory: 128Mi
    route:
      enabled: true
  rbac:
    defaultPolicy: ''
    policy: |
      g, system:cluster-admins, role:admin
    scopes: '[groups]'
  repo:
    resources:
      limits:
        cpu: 1000m
        memory: 1024Mi
      requests:
        cpu: 250m
        memory: 256Mi
  dex:
    openShiftOAuth: true
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  ha:
    enabled: false
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  redis:
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  controller:
    resources:
      limits:
        cpu: 2000m
        memory: 2048Mi
      requests:
        cpu: 250m
        memory: 1024Mi
```

-   Create the ArgoCD object:

```bash
$ oc apply -f argocd-crd.yaml
```