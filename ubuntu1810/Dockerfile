#
# http://github.com/bom4v/docker-images/tree/master/ubuntu1810
#
# Scala and SBT on Ubuntu 18.10 Cosmic with Java OpenJDK 8
# Script to download Oracle JDK: https://gist.github.com/n0ts/40dd9bd45578556f93e7
#
FROM docker.io/ubuntu:18.10
MAINTAINER Denis Arnaud <denis.arnaud_github at m4x dot org>
LABEL version="0.1"

# Environment
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV BOM4V_DIR $HOME/dev/bom4v

# Update the system
RUN apt-get update && apt-get -y install locales apt-utils && apt-get -y upgrade

# Generate the locales
RUN locale-gen $LANG

# Configure the time-zone
RUN apt-get -y install tzdata && echo "Europe/Paris" > /etc/timezone && \
	dpkg-reconfigure -f noninteractive tzdata

# Basic, Java and Scala packages
RUN apt-get -y update && \
	apt-get -y install sudo less htop net-tools wget curl screen git \
	keychain gawk bash-completion vim-nox emacs-nox apt-utils keyutils ftp \
	bzip2 gzip tar unzip lsb-release manpages \
	sqlite3 python python3 \
	man rake ruby-nokogiri

# Cleaning
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add bintray repository where the SBT binaries are published
RUN curl -sS https://bintray.com/sbt/rpm/rpm | \
	tee /etc/yum.repos.d/bintray-sbt-rpm.repo
RUN yum -y install sbt

# Environment for the `root` user
ADD resources/bashrc /root/.bashrc

# Java
ENV JDK_VERSION 8u192
ENV JDK_VERSION_MINOR b12
ENV JDK_RPM jdk-$JDK_VERSION-linux-x64.rpm
ENV JDK_RPM_TOKEN 750e1c8617c5452694857ad95c3ee230
ENV JDK_BASE_URL http://download.oracle.com/otn-pub/java/jdk
ENV JDK_RPM_URL $JDK_BASE_URL/$JDK_VERSION-$JDK_VERSION_MINOR/$JDK_RPM_TOKEN
ENV JDK_GETTER get_oracle_jdk_linux_x64.sh

# Scala
ENV SCALA_TAR_URL http://www.scala-lang.org/files/archive
ENV SCALA_VERSION 2.11.8
ENV SBT_VERSION 1.2.7
ENV SBT_OPTS -Xmx4G
ENV SBT_GBL $HOME/.sbt/1.2/global.sbt

# Download Java in a robust way
#ADD $JDK_GETTER /usr/local/bin
#RUN chmod a+x /usr/local/bin/$JDK_GETTER
#RUN /usr/local/bin/$JDK_GETTER

# Download Java in a ad hoc way (see https://stackoverflow.com/questions/10268583/downloading-java-jdk-on-linux-via-wget-is-shown-license-page-instead)
# and install it
RUN wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" $JDK_RPM_URL/$JDK_RPM && \
	yum -y install $JDK_RPM && rm -f $JDK_RPM

# Download and install Scala
RUN mkdir -p /opt && \
	wget $SCALA_TAR_URL/scala-$SCALA_VERSION.tgz && \
	tar xvf scala-$SCALA_VERSION.tgz && \
	mv scala-$SCALA_VERSION /opt && \
	ln -s /opt/scala-$SCALA_VERSION /opt/scala && \
	rm -f scala-$SCALA_VERSION.tgz

ENV PATH $PATH:/opt/scala/bin

# Create the `build` user (for the development activities)
RUN adduser --disabled-password --gecos "" build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Configure SSH
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh
RUN ssh-keyscan github.com > $HOME/.ssh/known_hosts
ADD resources/ssh-config $HOME/.ssh/config
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Set up the packaging environment for the `build` user
ADD resources/bashrc $HOME/.bashrc
ADD resources/gitconfig $HOME/.gitconfig
ADD resources/vimrc $HOME/.vimrc
RUN chmod 640 $HOME/.bashrc $HOME/.gitconfig $HOME/.vimrc
RUN chown -R build.build $HOME

# Switch to the 'build' user
WORKDIR $HOME
USER build

# Git prompt
RUN git clone https://github.com/magicmonty/bash-git-prompt.git ~/.bash-git-prompt --depth=1

# Run SBT once, at Docker build time, so that it does not download
# the dependencies at run time
RUN mkdir -p $BOM4V_DIR
WORKDIR $BOM4V_DIR
RUN sbt about

# Entry point
CMD ["/bin/bash"]

