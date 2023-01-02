#
# Dockerfile: https://github.com/bom4v/docker-images/tree/master/centos8/Dockerfile
# Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/javabigdata
# Usual Docker tag: infrahelpers/javabigdata:centos8
#
# Scala and SBT on CentOS 8 with Java OpenJDK 11
#
# Base image:
# * Hub/cloud: 'stream8' on
#   + Docker.io: https://hub.docker.com/r/tgagor/centos-stream 
#   + Quay.io: https://quay.io/repository/centos/centos?tab=tags
# * Dockerfile: https://github.com/tgagor/docker-centos/blob/master/stream8/Dockerfile
#
FROM quay.io/centos/centos:stream8

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"
LABEL version="0.1"

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV BOM4V_DIR $HOME/dev/bom4v
ENV RUBY_VERSION 3.2.0
ENV SBT_VERSION 1.8.0

# Import the Centos GPG key to prevent warnings
RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-Official &&\
  rpm --import https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Testing

# Add the repository for PowerTools
RUN dnf -y install 'dnf-command(config-manager)'
RUN dnf config-manager --set-enabled powertools

# Add EPEL support
RUN dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# Update of CentOS
RUN dnf -y clean all && \
    dnf -y upgrade || echo "Issue with upgrade; fine for now"

# Configure the locale
RUN dnf -y install glibc-locale-source glibc-langpack-en && \
    localedef -f UTF-8 -i en_US $LANG

# Environment for the `root` user
ADD resources/bashrc /root/.bashrc

# A few useful packages
RUN dnf -y install less htop net-tools which sudo man vim \
	git-all wget curl file bash-completion keyutils \
	zlib-devel bzip2-devel gzip unzip xz xz-devel \
	tar patch rpmconf yum-utils \
	make m4 autoconf automake libtool \
	langtable gcc gcc-c++ \
	redhat-rpm-config \
	openssl-devel libffi-devel libyaml-devel \
	readline-devel sqlite-devel mysql-devel \
	jq
RUN dnf -y clean all

# Install SBT
RUN dnf -y install https://repo.scala-sbt.org/scalasbt/rpm/sbt-$SBT_VERSION.rpm

# Java
RUN dnf -y install java-11-openjdk-devel maven

# Scala
ENV SCALA_TAR_URL https://www.scala-lang.org/files/archive
ENV SCALA_VERSION 2.12.17
ENV SBT_OPTS -Xmx4G
ENV SBT_GBL $HOME/.sbt/1.0/global.sbt

# Download and install Scala
RUN mkdir -p /opt && \
	curl -kLs $SCALA_TAR_URL/scala-$SCALA_VERSION.tgz \
	-o scala-$SCALA_VERSION.tgz && \
    tar xvf scala-$SCALA_VERSION.tgz && \
    mv scala-$SCALA_VERSION /opt && \
    ln -s /opt/scala-$SCALA_VERSION /opt/scala && \
    rm -f scala-$SCALA_VERSION.tgz

ENV PATH $PATH:/opt/scala/bin

# Create the `build` user (for the development activities)
RUN adduser build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Configure SSH
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh
RUN ssh-keyscan github.com > $HOME/.ssh/known_hosts
ADD resources/ssh-config $HOME/.ssh/config
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Set up the development environment for the `build` user
ADD resources/bashrc $HOME/.bashrc
ADD resources/gitconfig $HOME/.gitconfig
ADD resources/vimrc $HOME/.vimrc
RUN chmod 640 $HOME/.bashrc $HOME/.gitconfig $HOME/.vimrc
RUN chown -R build.build $HOME

# Switch to the `build` user
USER build
WORKDIR $BOM4V_DIR

# Ruby - https://www.linkedin.com/pulse/how-install-ruby-centos-7-amal-krishna-r
RUN git clone --depth 1 https://github.com/rbenv/rbenv.git $HOME/.rbenv
RUN git clone --depth 1 https://github.com/rbenv/ruby-build.git $HOME/.rbenv/plugins/ruby-build

# Install Ruby
RUN source $HOME/.bashrc && \
	rbenv install $RUBY_VERSION && \
	rbenv global $RUBY_VERSION

#
USER root
RUN chown -R build.build $HOME
USER build

# Install the nokogiri Gem
RUN source $HOME/.bashrc && gem install nokogiri --platform=ruby

# Run SBT once, at Docker build time, so that it does not download
# the dependencies at run time
RUN mkdir -p $BOM4V_DIR
WORKDIR $BOM4V_DIR
RUN sbt about

# Entry point
CMD ["/bin/bash"]

