FROM maven:3.9.11-eclipse-temurin-17

ARG YQ_VERSION=v4.48.1
ARG HELM_VERSION=v3.19.0
ARG KUBECTL_VERSION=v1.34.1

LABEL org.opencontainers.image.title="Jenkins Build Image" \
      org.opencontainers.image.description="Reusable CI image for Jenkins Kubernetes agents" \
      org.opencontainers.image.source="https://github.com/samael-1982/jenkins-build-image" \
      org.opencontainers.image.licenses="MIT"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      git \
      curl \
      wget \
      openssh-client \
      ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install yq
RUN wget -q \
    https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 \
    -O /usr/local/bin/yq && \
    chmod +x /usr/local/bin/yq

# Install kubectl
RUN curl -fsSL \
    https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl \
    -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl

# Install Helm
RUN wget -q \
    https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    tar -xzf helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    mv linux-amd64/helm /usr/local/bin/helm && \
    chmod +x /usr/local/bin/helm && \
    rm -rf linux-amd64 helm-${HELM_VERSION}-linux-amd64.tar.gz

# Verify installed tools
RUN git --version && \
    mvn -version && \
    kubectl version --client && \
    helm version && \
    yq --version

USER root