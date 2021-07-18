#
# http://github.com/bom4v/docker-images/tree/master/ubuntu1904
#
# Scala and SBT on Ubuntu 20.04 (Focal Fossal) with Java OpenJDK 11
#
FROM docker.io/ubuntu:20.04

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"
LABEL version="0.1"

# Environment
ENV container docker
ENV HOME /home/build
ENV DEBIAN_FRONTEND noninteractive
ENV BOM4V_DIR $HOME/dev/bom4v

# Locales
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG

# Update the system
RUN apt-get -qq update && \
    apt-get -y install apt-utils && \
    apt-get -y install libterm-ui-perl && \
    apt-get -y install locales && \
    apt-get -y upgrade

# Generate the locales
RUN locale-gen $LANG && \
    update-locale LANG=$LANG LANGUAGE=$LANGUAGE LC_ALL=$LC_ALL

# Configure the time-zone
RUN apt-get -qq update && \
    apt-get -y install tzdata && \
    echo "Europe/Paris" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

# GPG keys
RUN apt-get -y install gnupg2 && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 99E82A75642AC823

# Basic, Java and Scala packages
RUN apt-get -qq update && \
    apt-get -y install sudo less htop netcat net-tools wget curl screen git \
    apt-transport-https ca-certificates gnupg2 software-properties-common \
    keychain gawk bash-completion vim-nox emacs-nox apt-utils keyutils ftp \
    zip unzip gzip bzip2 xz-utils tar unrar p7zip-full \
    lsb-release man manpages \
    sqlite3 python python3 \
    maven openjdk-11-jdk \
    rake ruby-nokogiri

# Add SBT official repository
RUN echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee -a /etc/apt/sources.list.d/sbt.list
RUN curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add
RUN apt-get -qq update && \
    apt-get -y install sbt

# Cleaning
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Environment for the `root` user
ADD resources/bashrc /root/.bashrc
RUN git clone https://github.com/magicmonty/bash-git-prompt.git /root/.bash-git-prompt --depth=1

# Scala
ENV SCALA_TAR_URL http://www.scala-lang.org/files/archive
ENV SCALA_VERSION 2.12.14
ENV SBT_VERSION 1.4.9
ENV SBT_OPTS -Xmx4G
ENV SBT_GBL $HOME/.sbt/1.0/global.sbt

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
RUN git clone https://github.com/magicmonty/bash-git-prompt.git $HOME/.bash-git-prompt --depth=1

# Run SBT once, at Docker build time, so that it does not download
# the dependencies at run time
RUN mkdir -p $BOM4V_DIR
WORKDIR $BOM4V_DIR
RUN sbt about

# Entry point
CMD ["/bin/bash"]

