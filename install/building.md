# 构建

The Envoy build system uses Bazel. In order to ease initial building and for a quick start, we provide an Ubuntu 16 based docker container that has everything needed inside of it to build and *statically link* envoy, see [ci/README.md](https://github.com/envoyproxy/envoy/blob/master/ci/README.md).

In order to build manually, follow the instructions at [bazel/README.md](https://github.com/envoyproxy/envoy/blob/master/bazel/README.md).

## 要求

Envoy was initially developed and deployed on Ubuntu 14 LTS. It should work on any reasonably recent Linux including Ubuntu 16 LTS.

Building Envoy has the following requirements:

- GCC 5+ (for C++14 support).
- These [pre-built](https://github.com/envoyproxy/envoy/blob/master//ci/build_container/build_recipes) third party dependencies.
- These [Bazel native](https://github.com/envoyproxy/envoy/blob/master/bazel/repository_locations.bzl) dependencies.

Please see the linked [CI](https://github.com/envoyproxy/envoy/blob/master/ci/README.md) and [Bazel](https://github.com/envoyproxy/envoy/blob/master/bazel/README.md) documentation for more information on performing manual builds.

## 预构建的二进制文件

On every master commit we create a set of lightweight Docker images that contain the Envoy binary. We also tag the docker images with release versions when we do official releases.

- [envoyproxy/envoy](https://hub.docker.com/r/envoyproxy/envoy/tags/): Release binary with symbols stripped on top of an Ubuntu Xenial base.
- [envoyproxy/envoy-alpine](https://hub.docker.com/r/envoyproxy/envoy-alpine/tags/): Release binary with symbols stripped on top of a **glibc** alpine base.
- [envoyproxy/envoy-alpine-debug](https://hub.docker.com/r/envoyproxy/envoy-alpine-debug/tags/): Release binary with debug symbols on top of a **glibc** alpine base.

We will consider producing additional binary types depending on community interest in helping with CI, packaging, etc. Please open an [issue](https://github.com/envoyproxy/envoy/issues) in GitHub if desired.

## 修改 Envoy

If you’re interested in modifying Envoy and testing your changes, one approach is to use Docker. This guide will walk through the process of building your own Envoy binary, and putting the binary in an Ubuntu container.

- [Building an Envoy Docker image](sandboxes/local_docker_build.md)