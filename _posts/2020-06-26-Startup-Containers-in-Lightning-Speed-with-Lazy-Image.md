---
layout: post
title:  "Startup Containers in Lightning Speed"
date:   2017-12-21
desc: "Startup Containers in Lightning Speed with Lazy Image Distribution on Containerd"
keywords: "Jalpc,Jekyll,gh-pages,website,blog,easy"
categories: [HTML]
tags: [Jalpc,Jekyll]
icon: icon-html
---

# Startup Containers in Lightning Speed with Lazy Image Distribution on Containerd

> Introducing containerd non-core subproject Stargz Snapshotter

[![Kohei Tokunaga](https://miro.medium.com/fit/c/96/96/1*sxGlX8y4TMDiKLVkBBosyw.jpeg)](moz-extension://62849a71-f7f6-49e8-ae58-226e95d7425d/@ktokunaga.mail?source=post_page-----243d94522361----------------------)

Introducing containerd non-core subproject Stargz Snapshotter

![](https://miro.medium.com/max/60/1*ZsnXsWpeWAuhHBhyJAdgtQ.jpeg?q=20)

![](https://miro.medium.com/max/1600/1*ZsnXsWpeWAuhHBhyJAdgtQ.jpeg)

Picture by [irasutoya](https://www.irasutoya.com/)

Throughout container lifecycle, pulling image is one of the time-consuming steps. [Harter, et al.](https://www.usenix.org/node/194431) showed that,

> pulling packages accounts for 76% of container start time, but only 6.4% of that data is read

This has been affecting various kinds of workloads including cold-start of serverless functions, fetching base images during builds, CI/CD cycle, etc. In the community, workarounds are known but they still have unavoidable drawbacks as the following,

*   **Caching images**: but we cannot avoid performance penalty on cold start.
*   **Minimizing image size**: but big images are unavoidable for some use-cases including ML frameworks.

For solving this, several solutions have been proposed including [Slacker](https://www.usenix.org/node/194431), [CernVM-FS](https://cernvm.cern.ch/portal/filesystem) integration [with containerd](https://github.com/containerd/containerd/issues/2943), [Filegrain](https://github.com/akihirosuda/filegrain), [Microsoft Teleportation](https://stevelasker.blog/2019/10/29/azure-container-registry-teleportation/) and [Google CRFS](https://github.com/google/crfs).

Containerd, CNCF graduated container runtime project, started [**Stargz Snapshotter**](https://github.com/containerd/stargz-snapshotter) non-core subproject aiming to improve the pull performance. This is implemented as a plugin for containerd and enables it to _lazily_ pull container images leveraging [stargz image format by Google](https://github.com/google/crfs). The lazy pull here means containerd doesn’t download the entire image on `pull` operation but fetches necessary contents on-demand.

![](https://miro.medium.com/max/60/1*_nQ6rjj_Vz8tnx_kKdv83A.png?q=20)

![](https://miro.medium.com/max/3168/1*_nQ6rjj_Vz8tnx_kKdv83A.png)

containerd Stargz Snapshotter plugin for lazy pull

The following diagram shows the [HelloBench](https://github.com/Tintri/hello-bench)\-based benchmarking result of container startup operations, [measured on GitHub Actions and Docker Hub](https://github.com/containerd/stargz-snapshotter/actions?query=workflow%3ABenchmark+branch%3Amaster).

![](https://miro.medium.com/max/60/1*nq7IURKFDEnX0-zLB881dQ.png?q=20)

![](https://miro.medium.com/max/2000/1*nq7IURKFDEnX0-zLB881dQ.png)

Benchmarking result from [the project repository](https://github.com/containerd/stargz-snapshotter).

When we pull `legacy` images containerd needs to download the entire image for starting containers so time for `pull` takes accordingly. In the case of `stargz` images, containerd can lazily fetch them so `pull` takes shorter. But reading file induces downloading the file contents so the runtime performance `run` is lower than `legacy`. When we use the further optimized images `estargz`, this overhead for `run` can be mitigated. For more details and demo, please refer to the [GitHub repo](https://github.com/containerd/stargz-snapshotter).

Stargz snapshotter has the following features.

Standards compliant lazy pull
-----------------------------

Stargz snapshotter can lazily pull stargz images from [OCI](https://github.com/opencontainers/distribution-spec)/[Docker](https://docs.docker.com/registry/spec/api/) standard registries. And stargz images are compliant to [OCI](https://github.com/opencontainers/image-spec/)/[Docker](https://github.com/moby/moby/blob/master/image/spec/v1.2.md) image specifications so any runtimes can run them as well.

Private registries support
--------------------------

Stargz snapshotter supports authentication based on `~/.docker/config.json` as well as Kubernetes secrets. You can also configure mirror registries.

Kubernetes support
------------------

It supports [containerd’s CRI plugin](https://github.com/containerd/cri) it means we can run stargz snapshotter also on Kubernetes. [The repo contains a Dockerfile](https://github.com/containerd/stargz-snapshotter/blob/ad4ee3470aa34cdb3b9c2f8a8a131bb1dc75e697/Dockerfile) which can be used as a node image on KinD.

For using stargz snapshotter on kubernetes nodes, you need to run it as a daemon on each node and plug it into containerd. The following configuration is required for containerd (`/etc/containerd/config.toml`). We assume you are using containerd newer than at least commit `[d8506bf](https://github.com/containerd/containerd/commit/d8506bfd7b407dcb346149bcec3ed3c19244e3f1)`as a CRI runtime.

containerd configuration with stargz snapshotter

The stargz snapshotter repo [contains a Dockerfile](https://github.com/containerd/stargz-snapshotter/blob/ad4ee3470aa34cdb3b9c2f8a8a131bb1dc75e697/Dockerfile) as a node image which includes the above configuration. You can use it as a KinD node image like the following,

$ docker build -t stargz-kind-node [https://github.com/containerd/stargz-snapshotter.git](https://github.com/containerd/stargz-snapshotter.git)  
$ kind create cluster --name stargz-demo --image stargz-kind-node

Then you can create sample stargz pods on the cluster. In this example, we create a stargz-converted Node.js pod (`stargz/node:13.13.0-esgz`).

stargz version of Node.js sample pod

The pod creation lazily pulls `stargz/node:13.13.0-esgz` image from Docker Hub but it’s shorter than the original image(`library/node:13.13.0`).

$ kubectl --context kind-stargz-demo apply -f stargz-pod.yaml && kubectl get po nodejs -w  
$ kubectl --context kind-stargz-demo port-forward nodejs 8080:80 &  
$ curl 127.0.0.1:8080  
Hello World!

For further configuration including registry authentication, please [refer to the repo](https://github.com/containerd/stargz-snapshotter).

Stargz snapshotter is implemented as a combination of technologies. This section describes the overview of three of them.

[Stargz archive format](https://github.com/google/crfs)
-------------------------------------------------------

![](https://miro.medium.com/max/60/1*aba_56ZY6N-3Y-JuIwllpg.png?q=20)

![](https://miro.medium.com/max/4264/1*aba_56ZY6N-3Y-JuIwllpg.png)

traditional tar.gz vs stargz

For lazy image distribution, container runtimes need to download and extract file entries from layer blob, selectively. However, [OCI](https://github.com/opencontainers/image-spec/)/[Docker](https://github.com/moby/moby/blob/master/image/spec/v1.2.md) image specs define layer types as tar or tar.gz archives which require scanning the entire blob even for taking single file entry. Especially, when the layer is gzip-compressed, it is no longer seekable and we can’t fetch specific file entry anymore.

[Stargz](https://github.com/google/crfs) is an archive format proposed by Google. Initially, [this was discussed for optimizing builder image distribution in Go community](https://github.com/golang/go/issues/30829). Stargz stands for _Seekable tar.gz_ with which we can seek the archive and extract file entries selectively, without scanning the entire blob. For more details about stargz image format, please refer to [CRFS project repo](https://github.com/google/crfs). By combining this with HTTP Range Request supported by [OCI](https://github.com/opencontainers/distribution-spec)/[Docker](https://docs.docker.com/registry/spec/api/) registry specs, container runtimes can selectively fetch file entries from the registry. Additionally, stargz archive is still valid gzip so any container runtimes can treat stargz archived layers in the way just same as legacy “tar.gz” layers.

Futher optimization for stargz
------------------------------

![](https://miro.medium.com/max/60/1*mfr9bginI0ZenVqSgudlUw.png?q=20)

![](https://miro.medium.com/max/4408/1*mfr9bginI0ZenVqSgudlUw.png)

stargz vs eStargz

Stargz improves pull performance but it still has performance drawbacks for reading files which induce remotely fetching contents. For solving this, stargz snapshotter provides further optimization for images based on the workload.

Generally, container images are built with purpose and the _workloads_ are determined at the build. In many cases, a workload is defined in the Dockerfile using some parameters including entrypoint command, environment variables and user. The image converter command `ctr-remote images optimize` from stargz snapshotter project provides optimization for the performance of reading files that are most likely accessed in the defined workload. There are also some command options for specifying the image’s custom workload.

When converting the image, this command runs the specified workload in a sandboxed environment and profiles all file accesses. This command regards these files as likely accessed also in production. Then it sorts the archive entries by the accessed order and puts a landmark file at the end. Before running container, stargz snapshotter prefetches and precaches this range by single HTTP Range Request. This hopefully increases the cache hit rate for the specified workload and mitigates runtime overheads as shown in the benchmarking result.

Containerd remote snapshotter plugin
------------------------------------

![](https://miro.medium.com/max/60/1*PBcFBBSyk5uIdWexB3asZQ.png?q=20)

![](https://miro.medium.com/max/4198/1*PBcFBBSyk5uIdWexB3asZQ.png)

stargz snapshotter as a remote snapshotter plugin

Containerd has a clean and pluggable architecture and functionalities are implemented as plugins following the defined API. Users can use containerd with default plugins or with user-specific implementations, which gives users high extensibility. [AWS Firecracker is a well-known example which extends containerd for supporting microVMs](https://github.com/firecracker-microvm/firecracker-containerd).

Snapshotter is one of these plugins, which is used for storing extracted layers. During pulling an image, containerd extracts layers in the image and overlays them for preparing rootfs views called “snapshots” in the snapshotter. When containerd starts a container, it queries a snapshot to snapshotter and uses it as the container’s rootfs.

Containerd supports “remote” snapshotter which is a variant of snapshotter. This enables containerd to use remotely mounted snapshots without pulling the entire layer contents. Stargz snapshotter is a remote snapshotter implementation which understands stargz-archived layers. This provides remotely mounted stargz layers as snapshots and lazily download the actual file contents leveraging stargz format and HTTP Range Request.

We are continuously improving stargz snapshotter as well as seeking the adaptability for tools in container ecosystem. You can try this plugin from the demo in the repo. Contribution, integration and any kinds of feedbacks are always welcome!

NTT is looking for engineers who work in Open Source communities like Kubernetes & Docker projects. If you wish to work on such projects please do visit our recruitment page. To know more about NTT contribution towards open source projects please visit our Software Innovation Center page. We have a lot of maintainers and contributors in several open source projects. Our offices are located in the downtown area of Tokyo (Tamachi, Shinagawa) and Musashino.


[Source](https://medium.com/nttlabs/startup-containers-in-lightning-speed-with-lazy-image-distribution-on-containerd-243d94522361)