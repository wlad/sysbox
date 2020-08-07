<p align="center"><img alt="sysbox" src="./docs/figures/sysbox.png" width="800x" /></p>

## Introduction

**Sysbox** is an open-source container runtime (aka runc), originally
developed by [Nestybox](https://www.nestybox.com), that enables Docker
containers to act as virtual servers capable of running software such as
Systemd, Docker, and Kubernetes in them, easily and with proper isolation. This
allows you to use containers in new ways, and provides a faster, more efficient,
and more portable alternative to virtual machines in many scenarios.

Prior to Sysbox, running such software in a container required you to create
complex images, custom entrypoints, special volume mounts, and use very unsecure
privileged containers. With Sysbox, this is as simple as:

```
$ docker run --runtime=sysbox-runc -it some-image
```

In this container, you can now run Systemd, Docker, Kubernetes, etc., just like
you would on a physical host or virtual machine. You can launch inner containers
(and even inner privileged containers) with strong isolation from the
underlying host (via the Linux user-namespace). No more complex docker images or
docker run commands, and no need for unsecure privileged containers.

In order to do this, Sysbox uses many OS-virtualization features of the Linux
kernel and complements these with OS-virtualization techniques implemented in
user-space. These include using all Linux namespaces (in particular
the user-namespace), partial virtualization of procfs and sysfs, selective
syscall trapping, and more. Due to this, Sysbox requires a fairly recent Linux
kernel (see the [supported distros](#supported-distros) below).

Sysbox was forked from the OCI runc in early 2019, and has undergone significant
changes since then. It's written in Go, and it is currently composed of three
components: sysbox-runc, sysbox-fs, and sysbox-mgr. More on Sysbox's design can
be found in the [Sysbox user guide](docs/user-guide/design.md).

Sysbox sits below OCI-compatible container managers such as Docker / containerd,
allowing you to use these well known tools to deploy the containers. No need to
learn new tools.

The complete list of features is [here](#sysbox-features).

## Contents

-   [License](#license)
-   [Audience](#audience)
-   [System Containers](#system-containers)
-   [Supported Distros](#supported-distros)
-   [Host Requirements](#host-requirements)
-   [Installing Sysbox](#installing-sysbox)
-   [Using Sysbox](#using-sysbox)
-   [Documentation](#documentation)
-   [Sysbox Features](#sysbox-features)
-   [Integration with Container Managers & Orchestrators](#integration-with-container-managers--orchestrators)
-   [Sysbox is not Rootless Docker](#sysbox-is-not-rootless-docker)
-   [Sysbox does not use hardware virtualization](#sysbox-does-not-use-hardware-virtualization)
-   [Contributing](#contributing)
-   [Filing Issues](#filing-issues)
-   [Troubleshooting & Support](#troubleshooting--support)
-   [Roadmap](#roadmap)
-   [Relationship to Nestybox](#relationship-to-nestybox)
-   [Uninstallation](#uninstallation)
-   [Contact](#contact)
-   [Thank You](#thank-you)

## License

Sysbox is an open-source project, licensed under the Apache License, Version
2.0. See the [LICENSE](LICENSE) file for details.

## Audience

The Sysbox project is intended for anyone looking to experiment, invent, learn,
and build systems using system containers. It's cutting-edge OS virtualization,
and contributions are welcomed.

The Sysbox project is not meant for people looking for a commercially supported
solution. For such a solution, refer to the Sysbox Enterprise Edition
(sysbox-EE) in the [Nestybox website](https://www.nestybox.com). Sysbox-EE uses
Sysbox at its core, but complements it with enterprise-level features.

See [here](#relationship-to-nestybox) for more on the relationship between
the Sysbox open-source project and Nestybox.

## System Containers

We call the containers deployed by Sysbox **system containers**, to highlight the
fact that they can run not just micro-services (as regular containers do), but
also system software such as Docker, Kubernetes, Systemd, inner containers, etc.

More on system containers [here](docs/user-guide/concepts.md#system-container).

## Supported Distros

Sysbox relies on functionality that is currently only present in Ubuntu Linux.

See the [distro compatibility doc](docs/distro-compat.md) for information on what versions
of Ubuntu kernels are supported.

We plan to add support for more distros in the near future.

## Installing Sysbox

Before you can use Sysbox, you must first install it on your Linux machine.

There are two ways:

1) You can build it from source and install it manually. This is the best approach if you
   are looking for a deeper dive or if you want to contribute to Sysbox. See the
   [developer's guide](docs/developers-guide/README.md) for more on this.

or

2) You can download a packaged version from the [Nestybox website](https://www.nestybox.com).
   This is the easiest and best approach if you just want to use Sysbox.

## Using Sysbox

Once Sysbox is installed, you use it as follows:

```console
$ docker run --runtime=sysbox-runc --rm -it --hostname my_cont debian:latest
root@my_cont:/#
```

This launches a system container. It looks very much like a regular container,
but it's different under the hood. Within it can run application or system-level
software (systemd, dockerd, K8s) inside just as you would in a VM. The container
is strongly isolated via the Linux user-namespace.

The [Sysbox Quickstart Guide](docs/quickstart/README.md) has many usage examples.
You should start there to get familiarized with the use cases enabled by Sysbox.

Note that if you omit the `--runtime` option, Docker will use its default `runc`
runtime to launch regular containers (rather than system containers). It's
perfectly fine to run system containers launched with Docker + Sysbox alongside
regular Docker containers; they won't conflict and can co-exist side-by-side.

## Documentation

We strive to provide good documentation; it's a key component of the Sysbox project.

We have several documents to help you get started and get the best out of
Sysbox.

-   [Sysbox Quick Start Guide](docs/quickstart/README.md)

    -   Provides many examples for using system containers. New users
        should start here.

-   [Sysbox User Guide](docs/user-guide/README.md)

    -   Provides more detailed information on Sysbox features and design.

-   [Sysbox Distro Compatibility Doc](docs/distro-compat.md)

    -   Distro compatibility requirements.

## Sysbox Features

### OCI-based

-   Integrates with OCI compatible container managers (e.g., Docker).

-   Currently we only support Docker/containerd, but plan to add support for
    more managers / orchestrators (e.g., K8s) soon.

-   Sysbox is ~90% OCI-compatible. See [here](docs/user-guide/design.md#sysbox-oci-compatibility) for
    more on this.

### Systemd-in-Docker

-   Run Systemd inside a Docker container easily, without complex container configurations.

-   Enables you to containerize apps that rely on Systemd (e.g., legacy apps).

### Docker-in-Docker

-   Run Docker inside a container easily and without unsecure privileged containers.

-   Full isolation between the Docker inside the container and the Docker on the host.

### Kubernetes-in-Docker

-   Deploy Kubernetes (K8s) inside containers with proper isolation (no
    privileged containers), using simple Docker images and Docker run commands
    (no need for custom Docker images with tricky entrypoints).

-   Deploy directly with `docker run` commands for full flexibility, or using a
    higher level tool (e.g., such as [kindbox](https://github.com/nestybox/kindbox)).

### Strong container isolation

-   Root user in the system container maps to a fully unprivileged user on the host.

-   The procfs and sysfs exposed in the container are fully namespaced.

-   Programs running inside the system container (e.g., Docker, Kubernetes, etc)
    are limited to using the resources given to the system container itself.

-   Avoid the need for unsecure privileged containers.

### Fast & Efficient

-   Sysbox uses host resources optimally and starts containers in a few seconds.

### Inner Container Image Preloading

-   You can create a system container image that includes inner container
    images, with a simple Dockerfile or Docker commit.

Please see our [Roadmap](#roadmap) for a list of features we are working on.

## Integration with Container Managers & Orchestrators

Though Sysbox is OCI-based (and thus compatible with OCI container managers),
it's currently only tested with Docker / containerd.

In particular, we don't yet support using Kubernetes to deploy system containers
with Sysbox (though we [plan to](#roadmap)).

## Sysbox is not Rootless Docker

Sysbox often gets confused with [Rootless Docker](https://docs.docker.com/engine/security/rootless/), but it's in
fact very different.

Rootless Docker aims to run the Docker daemon in the host without root
privileges, to mitigate security risks. This however results in a number of
[limitations](https://docs.docker.com/engine/security/rootless/#known-limitations)
on what the Docker daemon can do.

Sysbox aims to create containers that can run any system software in them easily
and securely. The Docker on the host, as well as Sysbox, require root privileges
to make this possible. Within the containers however, you can run Docker and Kubernetes,
and they will only have privileges within the containers but none on the host.

What Rootless Docker and Sysbox have in common is that both use the Linux
user-namespace for isolation, but do so in different ways.

## Sysbox does not use hardware virtualization

Though the containers generated by Sysbox resemble virtual machines in some ways
(e.g., you can run as root, run multiple services, and deploy Docker and K8s
inside), Sysbox does not use hardware virtualization. It's purely an
OS-virtualization technology meant to create containers that can run
applications as well as system-level software, easily and securely.

This makes the containers created by Sysbox very fast, efficient, and
portable. Isolation wise, it's fair to say that they provide stronger isolation
than regular Docker containers (by virtue of using the Linux user-namespace),
but weaker isolation than VMs (by sharing the Linux kernel among containers).

## Contributing

We welcome contributions to Sysbox, whether they are small documentation changes,
bug fixes, or feature additions. Please see the [contribution guidelines](CONTRIBUTING.md)
and [developer's guide](docs/developers-guide/README.md) for more info.

## Filing Issues

We apologize for any problems in Sysbox or its documentation, and we appreciate
people filing issues that help us improve the software.

Please see the [contribution guidelines](CONTRIBUTING.md) for info on how
to report issues.

## Troubleshooting & Support

Refer to the [Troubleshooting document](docs/user-guide/troubleshoot.md)
and to the [issues](https://github.com/nestybox/sysbox/issues) for help.

Reach us at our [slack channel][slack] for any questions.

## Uninstallation

Prior to uninstalling Sysbox, make sure all system containers are removed.
There is a simple shell script to do this [here](scr/rm_all_syscont).

The method for uninstalling Sysbox depends on how you [installed it](#installing-sysbox).

1) If you used the packaged version from the Nestybox website, follow the
   uninstallation instructions in the associated documentation.

2) If you built it from source and installed it manually, follow [these instructions](docs/developers-guide/build.md)
   to uninstall it.

## Roadmap

The following is a list of features in the Sysbox roadmap.

We list these here so that our users can get a better idea of where we
are going and can give us feedback on which of these they like best
(or least).

Here is a short list; the Sysbox issue tracker has many more.

-   Support for more Linux distros.

-   Support for deploying system containers with Kubernetes.

-   More improvements to procfs and sysfs virtualization.

-   Continued improvements to container isolation.

-   Exposing host devices inside system containers with proper permissions.

## Relationship to Nestybox

Sysbox was initially developed by [Nestybox](https://www.nestybox.com), and Nestybox is
the main sponsor of the Sysbox open-source project.

Having said this, we encourage participation from the community to help evolve
and improve it, with the goal of increasing the use cases and benefits it
enables. External maintainers and contributors are welcomed.

Nestybox uses Sysbox as the core of it's Sysbox enterprise product (Sysbox-EE),
which consists of Sysbox plus proprietary features meant for enterprise use.

To ensure synergy between the Sysbox project and commercial entities such as
Nestybox, we use the following criteria when considering adding functionality to
Sysbox:

Any features that mainly benefit individual practitioners are made part of the Sysbox
open-source project. Any features that mainly address enterprise-level needs are
not part of the Sysbox open-source project.

The Sysbox project maintainers will make this determination on a feature by
feature basis, with total transparency. The goal is to create a balanced
approach that enables the Sysbox open-source community to benefit and thrive
while creating opportunities for commercial entities to create a healthy viable
business around the technology.

## Contact

Slack: [Nestybox Slack Workspace][slack]

We are there from Monday-Friday, 9am-5pm Pacific Time.

## Thank You

We thank you **very much** for using and/or contributing to Sysbox. We hope you
find it interesting and that it helps you use containers in new and more powerful
ways.

[slack]: https://nestybox-support.slack.com/join/shared_invite/enQtOTA0NDQwMTkzMjg2LTAxNGJjYTU2ZmJkYTZjNDMwNmM4Y2YxNzZiZGJlZDM4OTc1NGUzZDFiNTM4NzM1ZTA2NDE3NzQ1ODg1YzhmNDQ#/
