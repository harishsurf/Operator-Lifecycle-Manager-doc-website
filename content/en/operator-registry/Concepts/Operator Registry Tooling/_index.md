---
title: "Operator Registry Tooling"
linkTitle: "Operator Registry Tooling"
date: 2020-03-25
weight: 2
description: >
  [Operator-registry](https://github.com/operator-framework/operator-registry) project results in a collection of tools that in aggregate define a way of packaging and delivering operator manifests to Kubernetes clusters.
  This page describes these tools in detail.
---

{{% alert title="Warning" color="warning" %}}
This section is under construction. 
{{% /alert %}}


`opm` (Operator Package Manager) is a tool that is used to generate and interact with operator-registry catalogs, both the underlying databases (generally referred to as the registry) and their images (the index).
 
This is divided into two main commands: 
* `opm registry` which is used to initialize, update and serve an API of the underlying database of manifests and references.
* `opm index` which is used to interact with an OCI container runtime to generate the registry database and package it in a container image.
