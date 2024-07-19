FROM us-docker.pkg.dev/fairwinds-ops/oss/pluto:v5 as pluto
FROM quay.io/fairwinds/polaris:9 as polaris

FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ARG TARGETARCH
RUN yes | unminimize && \
    apt-get update && \
    apt-get install -yqq \
        ca-certificates \
        gnupg \
        lsb-release \
        apt-transport-https \
        curl \
        wget \
        unzip \
        git \
        vim \
        man \
        iputils-ping \
        inetutils-telnet \
        netcat \
        iproute2 \
        bash-completion \
        jq \
        yamllint \
        dnsutils \
        gcc \
        zlib1g-dev \
        postgresql-client

# Install GraalVM
ARG JAVA_VERSION=21
ARG GRAAL_VERSION=21.0.2
RUN mkdir -p /usr/lib/jvm && \
    cd /usr/lib/jvm && \
    curl -fsSL https://github.com/graalvm/graalvm-ce-builds/releases/download/jdk-${GRAAL_VERSION}/graalvm-community-jdk-${GRAAL_VERSION}_linux-$(if [ $TARGETARCH = "arm64" ]; then echo aarch64; else echo x64; fi)_bin.tar.gz | tar xzf - && \
    mv graalvm-community-openjdk-* graalvm-community-openjdk-$JAVA_VERSION
ENV JAVA_HOME=/usr/lib/jvm/graalvm-community-openjdk-$JAVA_VERSION
ENV PATH=$PATH:$JAVA_HOME/bin

# https://docs.docker.com/engine/install/ubuntu/
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    echo "deb [arch=${TARGETARCH} signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
    tee /etc/apt/sources.list.d/docker.list

# https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg && \
    echo "deb [arch=${TARGETARCH} signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | \
    tee /etc/apt/sources.list.d/kubernetes.list

# https://helm.sh/docs/intro/install/#from-apt-debianubuntu
RUN curl -fsSL https://baltocdn.com/helm/signing.asc | \
    gpg --dearmor -o /etc/apt/keyrings/helm.gpg && \
    echo "deb [arch=${TARGETARCH} signed-by=/etc/apt/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | \
    tee /etc/apt/sources.list.d/helm-stable-debian.list

# Additional binaries from those repositories
RUN apt-get update && \
    apt-get install -yqq \
        docker-ce-cli \
        docker-compose-plugin \
        kubectl \
        helm

# kubectl completions
RUN echo "source <(kubectl completion bash)" | tee /etc/bash_completion.d/kubectl
RUN echo "source /etc/profile.d/bash_completion.sh" >> /etc/bash.bashrc

# https://docs.gradle.org/8.7/userguide/installation.html#ex-installing-manually
ARG GRADLE_VERSION=8.7
RUN mkdir -p /opt/gradle && \
    curl -fsSL https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -o /tmp/gradle.zip && \
    unzip -d /opt/gradle /tmp/gradle.zip && \
    rm /tmp/gradle.zip
ENV GRADLE_HOME=/opt/gradle/gradle-${GRADLE_VERSION}
ENV PATH=$PATH:$GRADLE_HOME/bin

# Gradle completions
ARG GRADLE_COMPLETION_VERSION=1.4.1
RUN curl -fsSL https://raw.githubusercontent.com/gradle/gradle-completion/v${GRADLE_COMPLETION_VERSION}/gradle-completion.bash -o /etc/bash_completion.d/gradle

# Quarkus CLI
ARG QUARKUS_VERSION=3.9.1
RUN mkdir -p /opt/quarkus && \
    curl -fsSL https://github.com/quarkusio/quarkus/releases/download/${QUARKUS_VERSION}/quarkus-cli-${QUARKUS_VERSION}.tar.gz | tar xzvf - -C /opt/quarkus
ENV QUARKUS_HOME=/opt/quarkus/quarkus-cli-${QUARKUS_VERSION}
ENV PATH=$PATH:$QUARKUS_HOME/bin
RUN echo "source <(quarkus completion)" | tee /etc/bash_completion.d/quarkus

# Install yq
ARG YQ_VERSION=4.30.5
RUN wget https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${TARGETARCH} -O /usr/bin/yq && \
    chmod 755 /usr/bin/yq

# Install pluto and polaris
COPY --from=polaris /usr/local/bin/polaris /usr/local/bin/polaris
COPY --from=pluto /pluto /usr/local/bin/pluto
