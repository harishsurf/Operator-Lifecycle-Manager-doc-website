---
title: "opm index"
linkTitle: "opm index"
date: 2020-03-25
weight: 2
description: >
    opm index is a wrapper for opm registry that abstracts the underlying database interaction to instead make it easier to speak about the container images that are actually shipped to clusters directly. 
---


### add

Index add works much the same way as registry add. For example:

```bash
opm index add --bundles quay.io/operator-framework/operator-bundle-prometheus:0.14.0 --tag quay.io/operator-framework/monitoring-index:1.0.0
```

Just like `opm registry add`, this command pulls a given container bundle and attempts to put it into a registry. The real difference is that the result is more than just a database file. By default, this command actually builds a container image and, looking at the `--tag` flag, will tag the output image as `quay.io/operator-framework/monitoring-index:1.0.0`. The resulting image has the database and the opm binary in it and, when run, calls the `registry serve` command on the database that was generated.

Just like registry add command, the updates are cumulative. In this case, rather than pointing at a database file, we can use the `--from-index` flag to specify a previous index to build off of a previous registry:

```bash
opm index add --bundles quay.io/operator-framework/operator-bundle-prometheus:0.15.0 --from-index quay.io/operator-framework/monitoring:1.0.0 --tag quay.io/operator-framework/monitoring:1.0.1
```

This results in a fresh image that includes the updated prometheus operator in the prometheus package's update graph.

At a high level, this command operates by wrapping `registry add` around some additional interaction with pulling and building container images. To that end, the last thing it does is actually shell out to a container CLI tool to build the resulting container (by default, `podman build`). It does this by generating a dockerfile and then passing that file to the shell command. For example:

```dockerfile
FROM quay.io/operator-framework/upstream-registry-builder AS builder

FROM scratch
LABEL operators.operatorframework.io.index.database.v1=./index.db
COPY database ./
COPY --from=builder /build/bin/opm /opm
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe
EXPOSE 50051
ENTRYPOINT ["/opm"]
CMD ["registry", "serve", "--database", "index.db"]
```

Of note here is that we use a builder image to get the latest upstream released version of opm in order to call `opm registry serve` to host the gRPC API. If a developer or CI system would prefer to point to a different version of `opm` to serve their operator (perhaps one in a private release or a fork) then they just need to deliver their own version in a container and then use the `--binary-image` command. ex:

```bash
opm index add --bundles quay.io/operator-framework/operator-bundle-prometheus:0.14.0 --tag quay.io/operator-framework/monitoring-index:1.0.0 --binary-image quay.io/$user/my-opm-source
```

This will update the above dockerfile and replace the builder image with the image specified in the `--binary-image` flag.

We are aware of the fact that, in many cases, users will want to make other changes to this dockerfile (adding additional labels, adding other binaries for metrics, using a different port, etc.). For these more complex use cases, we have added the `--generate` and `--out-dockerfile` flags. Adding `--generate` will skip the container build command entirely and instead write a Dockerfile to the local filesystem. By default this file is called `index.Dockerfile` and is put in the directory you run `opm` from. If you want to rename this generated dockerfile and write it somewhere else, just specify the `--out-dockerfile` flag:

```bash
opm index add --bundles quay.io/operator-framework/operator-bundle-prometheus:0.14.0 --generate --out-dockerfile "my.Dockerfile"
```

Running this command will still generate the updated registry database, but it will store it locally and additionally write `my.Dockerfile` which can be modified as needed.

### rm

Like `opm registry rm`, this command will remove all versions an entire operator package from the index and results in a container image that does not include that package. It supports virtually all of the same options and flags as `opm index add` with the exception of replacing `--bundles` with `--operators`. Ex:

```bash
opm index rm --operators prometheus --tag quay.io/operator-framework/monitoring-index:1.0.2 --binary-image quay.io/$user/my-opm-source
```

This will result in the tagged container image `quay.io/operator-framework/monitoring-index:1.0.2` with a registry that no longer contains the `prometheus` operator at all.

### export

`opm index export` will export a package from an index image into a directory. The format of this directory will match the appregistry manifest format: containing all versions of the package in the index along with a `package.yaml` file. This command takes an `--index` flag that points to an index image, a `--package` flag that states a package name, an optional `--download-folder` as the export location (default is `./downloaded`), and just as the other index commands it takes a `--container-tool` flag. Ex:

```bash
opm index export --index="quay.io/operator-framework/monitoring:1.0.0" --package="prometheus" -c="podman"
```

This will result in the following `downloaded` folder directory structure:

```bash
downloaded
├── 0.14.0
│   ├── alertmanager.crd.yaml
│   ├── prometheus.crd.yaml
│   ├── prometheusoperator.0.14.0.clusterserviceversion.yaml
│   ├── prometheusrule.crd.yaml
│   └── servicemonitor.crd.yaml
├── 0.15.0
│   ├── alertmanager.crd.yaml
│   ├── prometheus.crd.yaml
│   ├── prometheusoperator.0.15.0.clusterserviceversion.yaml
│   ├── prometheusrule.crd.yaml
│   └── servicemonitor.crd.yaml
├── 0.22.2
│   ├── alertmanager.crd.yaml
│   ├── prometheus.crd.yaml
│   ├── prometheusoperator.0.22.2.clusterserviceversion.yaml
│   ├── prometheusrule.crd.yaml
│   └── servicemonitor.crd.yaml
└── package.yaml
```

which can be pushed to appregistry.


{{% alert title="Warning" color="warning" %}}
The appregistry format is being deprecated in favor of the new index image and image bundle format.
{{< /alert >}}