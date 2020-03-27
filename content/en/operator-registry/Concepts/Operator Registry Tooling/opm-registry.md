---
title: "opm registry"
linkTitle: "opm registry"
date: 2020-03-25
weight: 1
description: >
  opm registry generates and updates registry database objects.
---

{{% alert title="Warning" color="warning" %}}
These pages are under construction. TODO: Verify Content
{{% /alert %}}


### add

First, let's look at adding a version of an operator bundle to a registry database. For example:

```bash
opm registry add -b "quay.io/operator-framework/operator-bundle-prometheus:0.14.0" -d "test-registry.db"
```

Dissecting this command, we called `opm registry add` to pull a container image that includes the manifests for the 0.14.0 version of the `prometheus` operator. We then unpacked those manifests from the container image and attempted to insert them into the registry database `test-registry.db`. Since that database file didn't currently exist on disk, the database was initialized first and then prometheus 0.14.0 was added to the empty database.

Now imagine that the 0.15.0 version of the `prometheus operator` was just released. We can add that operator to our existing database by calling add again and pointing to the new container image:

```bash
opm registry add -b "quay.io/operator-framework/operator-bundle-prometheus:0.15.0" -d "test-registry.db"
```

Great! The existing `test-registry.db` file is updated. Now we have a registry that contains two versions of the operator and defines an update graph that, when added to a cluster, will signal to the Operator Lifecycle Manager that if you have already installed version `0.14.0` that `0.15.0` can be used to upgrade your installation.

### rm

`opm` also currently supports removing entire packages from a registry. For example:

```bash
opm registry rm -o "prometheus" -d "test-registry.db"
```

Calling this on our existing test registry removes all versions of the prometheus operator entirely from the database.

### serve

`opm` also includes a command to connect to an existing database and serve a `gRPC` API that handles requests for data about the registry:

```bash
opm registry serve -d "test-registry.db" -p 50051
```
