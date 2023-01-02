Docker images to support big data development on Scala/Java stacks
==================================================================

[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/infrahelpers/javabigdata)](https://hub.docker.com/repository/docker/infrahelpers/javabigdata/general)
[![Docker Repository on Quay](https://quay.io/repository/bigdatadevelopment/base/status "Docker Repository on Quay")](https://quay.io/repository/bigdatadevelopment/base)

# Introduction
[That project](https://github.com/bom4v/docker-images)
produces Docker images, hosted on [dedicated
public Docker Cloud site](https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/javabigdata).
Those Docker images are intended to bring Linux-based ready-to-use environment
for Scala/Java big data developers. Both programming languages are relying
on the Java Virtual Machine (JVM) ecosystem.
[Apache Spark](http://spark.apache.org) is featured in particular.
SBT-built artefacts can for instance be launched on a stand-alone Spark version.

The supported Linux distributions are:
- [CentOS 7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7)
- [CentOS 8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS8)
- [Ubuntu 18.04 LTS (Bionic Beaver)](http://releases.ubuntu.com/18.04/)
- [Ubuntu 20.04 LTS (Focal Fossal)](http://releases.ubuntu.com/20.04/)
- [Debian 10 (Buster)](https://www.debian.org/releases/buster/)
- [Debian 9 (Stretch)](https://www.debian.org/releases/stretch/)

Every time some changes are committed on the
[project's GitHub repository](https://github.com/bom4v/docker-images),
the
[Docker images are automatically rebuilt](https://hub.docker.com/repository/docker/infrahelpers/javabigdata/builds)
and pushed onto Docker Cloud.

When some more components are needed, which may be of interest to other
Scala/Java big data developers, the Docker image may be amended so as to add
those extra components.
The preferred way to propose amendment of the Docker image is through
[pull requests on the GitHub project](https://github.com/bom4v/docker-images/pulls).
Once the pull request has been merged, _i.e._, once the `Dockerfile` amendment
has been
[committed in GitHub](https://github.com/bom4v/docker-images/commits/master),
Docker Cloud then rebuilds the corresponding Docker images, which become
available for every one to use.

# Images on Docker Cloud
* [Docker Cloud dashboard](https://hub.docker.com/repository/docker/infrahelpers/javabigdata/general)

# Using the pre-built development images
* Start the Docker container featuring the target Linux distribution
  (`<linux-distrib>` may be one of `centos8`, `centos7`,
  `debian11`, `debian10`,
  `ubuntu2204`, `ubuntu2004` or `ubuntu1804`):
```bash
$ docker pull infrahelpers/javabigdata:<linux-distrib>
$ docker run --rm -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it infrahelpers/javabigdata:<linux-distrib>
[build@5..0 dev]$ 
```

* Setup the user name and email address for Git:
```bash
[build@5..0 dev]$ git config --global user.name "Firstname Lastname"
[build@5..0 dev]$ git config --global user.email "email@example.com"
```

* Clone some Scala/Java-based project (_e.g._,
  [BOM4V meta-models](http://github.com/bom4v/metamodels)
  is used as an example here):
```bash
[build@5..0 dev]$ git clone https://github.com/bom4v/metamodels.git
Cloning into 'metamodels...
remote: Enumerating objects: 44, done.
remote: Counting objects: 100% (44/44), done.
remote: Compressing objects: 100% (35/35), done.
Receiving objects: 100% (5813/5813), 61.53 MiB | 211.00 KiB/s, done.
remote: Total 5813 (delta 12), reused 19 (delta 8), pack-reused 5769
Resolving deltas: 100% (3665/3665), done.
[build@5..0 dev]$ cd metamodels
[build@5..0 metamodels (master)]$ 
```

* Do some development:
```bash
[build@5..0 metamodels (trunk)]$ 
```

# Customize a Docker Image
The images may be customized, and pushed to Docker Cloud;
`<linux-distrib>` may be one of `centos8`, `centos7`,
  `debian11`, `debian10`,
  `ubuntu2204`, `ubuntu2004` or `ubuntu1804`:
```bash
$ mkdir -p ~/dev
$ cd ~/dev
$ git clone https://github.com/bom4v/docker-images.git jvm-docker-images
$ cd jvm-docker-images
$ vi <linux-distrib>/Dockerfile
$ docker build -t infrahelpers/javabigdata:<linux-distrib> <linux-distrib>/
$ docker run --rm -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it infrahelpers/javabigdata:<linux-distrib>
[build@9..d jvm-docker-images]$ exit
$ docker push infrahelpers/javabigdata:<linux-distrib>
```

# TODO
For any of the following features, an issue may be open
[on GitHub](https://github.com/bom4v/docker-images/issues):
1. Support other Linux distributions, for instance Ubuntu 16.04 LTS
   or Fedora (_e.g._, `fedora`)
2. Automate regular rebuilds (_e.g._, once a month for CentOS or Ubuntu)


