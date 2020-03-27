---
title: "Getting Started"
linkTitle: "Getting Started"
date: 2020-03-25
weight: 2
description: >
    Package and deliver operator manifests to Kubernetes cluster using [Operator-Registry] 
---

{{% alert title="Warning" color="warning" %}}
These pages are under construction. **TODO: Verify each section**
{{% /alert %}}


## Prerequisites

- [git](https://git-scm.com/downloads)
- [go](https://golang.org/dl/) version `v1.12+`.
- [docker](https://docs.docker.com/install/) version `17.03`+.
  - Alternatively [podman](https://github.com/containers/libpod/blob/master/install.md) `v1.2.0+` or [buildah](https://github.com/containers/buildah/blob/master/install.md) `v1.7+`
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) `v1.11.3+`.
- Access to a Kubernetes `v1.11.3+` cluster.

## Installation

### `opm` (Operator Package Manager)

opm (Operator Package Manager) is a tool that is used to generate and interact with operator-registry catalogs, both the underlying databases (generally referred to as the registry) and their images (the index).

In order to use `opm` CLI, follow the `opm` build instruction:

1. Clone the operator registry repository:

```bash
$ git clone https://github.com/operator-framework/operator-registry
```

2. Build `opm` binary using this command:

```bash
$ go build ./cmd/opm/
```

Now, a binary named `opm` is now built in current directory and ready to be used.
