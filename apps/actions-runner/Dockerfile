ARG VERSION=latest
FROM ghcr.io/actions/actions-runner:${VERSION}
ARG TARGETARCH

ENV HOMEBREW_NO_ANALYTICS=1 \
    HOMEBREW_NO_ENV_HINTS=1 \
    HOMEBREW_NO_INSTALL_CLEANUP=1 \
    DEBCONF_NONINTERACTIVE_SEEN=true \
    DEBIAN_FRONTEND="noninteractive" \
    APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn

ENV \
    # renovate: datasource=github-releases depName=cli/cli
    GH_CLI_VERSION="v2.85.0"

USER root

RUN \
    apt-get -qq update && apt-get -y dist-upgrade \
    && \
    apt-get -qq install -y --no-install-recommends --no-install-suggests \
        ca-certificates \
        apt-utils \
        curl \
        jo \
        moreutils \
        wget \
        zstd \
    && \
    case "${TARGETARCH}" in \
        'amd64') apt-get -qq install -y --no-install-recommends --no-install-suggests gcc ;; \
    esac \
    && \
    curl -fsSL "https://github.com/mikefarah/yq/releases/latest/download/yq_linux_${TARGETARCH}" -o /usr/local/bin/yq \
        && chmod +x /usr/local/bin/yq \
    && \
    mkdir -p -m 755 /etc/apt/keyrings \
        && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        && cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
        && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
        && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
        && apt update \
        && apt install gh -y \
    && \
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/${TARGETARCH}/kubectl" \
        && chmod +x ./kubectl \
        && mv ./kubectl /usr/local/bin/kubectl \
    && \
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \
        && chmod +x get_helm.sh && ./get_helm.sh \
        && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && \
    chown -R runner:runner /usr/local/bin

USER runner

RUN \
    case "${TARGETARCH}" in \
        'amd64') /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)" ;; \
    esac
