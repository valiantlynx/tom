# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.224.3/containers/ubuntu/.devcontainer/base.Dockerfile

# [Choice] Ubuntu version (use hirsuite or bionic on local arm64/Apple Silicon): hirsute, focal, bionic
ARG VARIANT="hirsute"
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT}

# [Optional] Uncomment this section to install additional OS packages.
 RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
     && dpkg --print-architecture\
     && dpkg --print-foreign-architectures \
     && dpkg --add-architecture i386\
     && dpkg --print-foreign-architectures
 RUN apt-get update \
     && apt-get -y install --no-install-recommends autoconf \
     chrpath fluid imagemagick libglib2.0-dev libtool \
     subversion xsltproc gawk dh-autoreconf pkg-config \
     libglib2.0-dev libc6:i386 libncurses5:i386 libstdc++6:i386 cmake
 RUN apt-get update
