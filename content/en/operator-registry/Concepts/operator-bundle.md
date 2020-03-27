---
title: "Operator Bundle"
linkTitle: "Operator Bundle"
date: 2020-03-25
weight: 1
description: >
  An `Operator Bundle` is a container image that stores Kubernetes manifests and metadata associated with an operator.
---

### Overview

The operator manifests refers to a set of Kubernetes manifest(s) the defines the deployment and RBAC model of the operator. The operator metadata on the other hand are, but not limited to:

* Information that identifies the operator, its name, version etc.
* Additional information that drives the UI:
    * Icon
    * Example CR(s)
* Channel(s)
* API(s) provided and required.
* Related images.

An `Operator Bundle` is built as a scratch (non-runnable) container image that contains operator manifests and specific metadata in designated directories inside the image. Then, it can be pushed and pulled from an OCI-compliant container registry. Ultimately, an operator bundle will be used by Operator Registry and Operator-Lifecycle-Manager (OLM) to install an operator in OLM-enabled clusters.

### Bundle Manifest Format

The standard bundle format requires two directories named `manifests` and `metatdata`. The `manifests` directory is where all operator manifests are resided including ClusterServiceVersion (CSV), CustomResourceDefinition (CRD) and other supported Kubernetes types. The `metadata` directory is where operator metadata is located including `annotations.yaml` which contains additional information such as package name, channels and media type.

Below is the directory layout of an operator bundle inside a bundle image:
```bash
$ tree
/
├── manifests
│   ├── etcdcluster.crd.yaml
│   └── etcdoperator.clusterserviceversion.yaml
└── metadata
    └── annotations.yaml
```

{{< alert title="Note" >}}The names of manifests and metadata directories must match the bundle annotations that are provided in `annotations.yaml` file. Currently, those names are set to `manifests` and `metadata`.{{< /alert >}}


### Bundle Annotations

We use the following labels to annotate the operator bundle image.
* The label `operators.operatorframework.io.bundle.mediatype.v1` reflects the media type or format of the operator bundle. It could be helm charts, plain Kubernetes manifests etc.
* The label `operators.operatorframework.io.bundle.manifests.v1 `reflects the path in the image to the directory that contains the operator manifests. This label is reserved for the future use and is set to `manifests/` for the time being.
* The label `operators.operatorframework.io.bundle.metadata.v1` reflects the path in the image to the directory that contains metadata files about the bundle. This label is reserved for the future use and is set to `metadata/` for the time being.
* The `manifests.v1` and `metadata.v1` labels imply the bundle type:
    * The value `manifests.v1` implies that this bundle contains operator manifests.
    * The value `metadata.v1` implies that this bundle has operator metadata.
* The label `operators.operatorframework.io.bundle.package.v1` reflects the package name of the bundle.
* The label `operators.operatorframework.io.bundle.channels.v1` reflects the list of channels the bundle is subscribing to when added into an operator registry
* The label `operators.operatorframework.io.bundle.channel.default.v1` reflects the default channel an operator should be subscribed to when installed from a registry

The labels will also be put inside a YAML file, as shown below.

*annotations.yaml*
```yaml
annotations:
  operators.operatorframework.io.bundle.mediatype.v1: "registry+v1"
  operators.operatorframework.io.bundle.manifests.v1: "manifests/"
  operators.operatorframework.io.bundle.metadata.v1: "metadata/"
  operators.operatorframework.io.bundle.package.v1: "test-operator"
  operators.operatorframework.io.bundle.channels.v1: "beta,stable"
  operators.operatorframework.io.bundle.channel.default.v1: "stable"
```

{{< alert title="Note" >}}
* In case of a mismatch, the `annotations.yaml` file is authoritative because the on-cluster operator-registry that relies on these annotations has access to the yaml file only.
* The potential use case for the `LABELS` is - an external off-cluster tool can inspect the image to check the type of a given bundle image without downloading the content.
* The annotations for bundle manifests and metadata are reserved for future use. They are set to be `manifests/` and `metadata/` for the time being.
{{< /alert >}}

### Bundle Dockerfile

This is an example of a `Dockerfile` for operator bundle:
```Dockerfile
FROM scratch

# We are pushing an operator-registry bundle
# that has both metadata and manifests.
LABEL operators.operatorframework.io.bundle.mediatype.v1=registry+v1
LABEL operators.operatorframework.io.bundle.manifests.v1=manifests/
LABEL operators.operatorframework.io.bundle.metadata.v1=metadata/
LABEL operators.operatorframework.io.bundle.package.v1=test-operator
LABEL operators.operatorframework.io.bundle.channels.v1=beta,stable
LABEL operators.operatorframework.io.bundle.channel.default.v1=stable

ADD test/*.yaml /manifests
ADD test/metadata/annotations.yaml /metadata/annotations.yaml
```