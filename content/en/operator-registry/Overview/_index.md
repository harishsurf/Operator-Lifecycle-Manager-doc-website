---
title: "Overview"
linkTitle: "Overview"
weight: 1
description: >
    This page introduces you to Operator Registry and what you can achieve with it
---

{{% alert title="Warning" color="warning" %}}
These pages are under construction. TODO: Some of the sections below needs to be updated.
{{% /alert %}}


## What is Operator-Registry?

[Operator Registry](https://github.com/operator-framework/operator-registry) runs in a Kubernetes or OpenShift cluster to provide operator catalog data to [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager).


Operator Registry provides the following binaries:

 * `opm`, which generates and updates registry databases as well as the index images that encapsulate them.
 * `initializer`, which takes as an input a directory of operator manifests and outputs a sqlite database containing the same data for querying.
 * `registry-server`, which takes a sqlite database loaded with manifests, and exposes a gRPC interface to it.
 * `configmap-server`, which takes a kubeconfig and a configmap reference, and parses the configmap into the sqlite database before exposing it via the same interface as `registry-server`.
 
And libraries:
 
 * `pkg/client` - providing a high-level client interface for the gRPC api.
 * `pkg/api` - providing low-level client libraries for the gRPC interface exposed by `registry-server`.
 * `pkg/registry` - providing basic registry types like Packages, Channels, and Bundles.
 * `pkg/sqlite` - providing interfaces for building sqlite manifest databases from `ConfigMap`s or directories, and for querying an existing sqlite database.
 * `pkg/lib` - providing external interfaces for interacting with this project as an api that defines a set of standards for operator bundles and indexes.
 * `pkg/containertools` - providing an interface to interact with and shell out to common container tooling binaries (if installed on the environment)

## Why do I want Operator Registry?


## What is it good for?
 
What types of problems does your project solve? What are the benefits of using it?

## What is it not good for?
 
For example, point out situations that might intuitively seem suited for your project, but aren't for some reason. Also mention known limitations, scaling issues, or anything else that might let your users know if the project is not for them.

## What is it *not yet* good for?

Highlight any useful features that are coming soon.

## Where should I go next?

Give your users next steps from the Overview. For example:

* [Getting Started](/operator-registry/getting-started/): Get started with project
* [Examples](/operator-registry/examples/): Check out some example code!