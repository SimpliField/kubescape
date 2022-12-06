VERSION 0.6

ARG DOCKER_BUILD_IMAGE_PREFIX="europe-west1-docker.pkg.dev/simplifield-platform-dev/tools"
ARG GCLOUD_SDK_VERSION="410.0.0"

build-all:
    BUILD +kubescape

builder:
    FROM golang:1.18-alpine

    ARG image_version
    ARG client

    ENV RELEASE=$image_version
    ENV CLIENT=$client

    ENV GO111MODULE=

    ENV CGO_ENABLED=1

    # Install required python/pip
    ENV PYTHONUNBUFFERED=1
    RUN apk add --update --no-cache python3 gcc make git libc-dev binutils-gold cmake pkgconfig && ln -sf python3 /usr/bin/python
    RUN python3 -m ensurepip
    RUN pip3 install --no-cache --upgrade pip setuptools

    WORKDIR /work
    COPY . .

    # install libgit2
    RUN rm -rf git2go && make libgit2

    # build kubescape cmd
    WORKDIR /work
    RUN python build.py

    SAVE ARTIFACT /work/build/ubuntu-latest/kubescape

    RUN /work/build/ubuntu-latest/kubescape download artifacts -o /work/artifacts

    SAVE ARTIFACT /work/artifacts

kubescape:
    FROM google/cloud-sdk:${GCLOUD_SDK_VERSION}-alpine
    RUN gcloud components install gke-gcloud-auth-plugin && gcloud components install kubectl
    RUN addgroup -S ks && adduser -S ks -G ks

    COPY +builder/artifacts /home/ks/.kubescape

    RUN chown -R ks:ks /home/ks/.kubescape

    USER ks

    WORKDIR /home/ks

    COPY +builder/kubescape /usr/bin/kubescape
    
    ARG DOCKER_BUILD_TAG="dev"
    SAVE IMAGE --push $DOCKER_BUILD_IMAGE_PREFIX/sf-kubescape:$DOCKER_BUILD_TAG
