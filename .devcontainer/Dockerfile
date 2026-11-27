FROM --platform=linux/amd64 debian:12.15

ENV DEBIAN_FRONTEND=noninteractive

ARG CORP_CA_B64=""

# Install certificate tooling before adding the optional corporate CA.
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates \
    && if [ -n "$CORP_CA_B64" ]; then \
         printf '%s' "$CORP_CA_B64" \
           | base64 -d \
           > /usr/local/share/ca-certificates/corp-ca.crt; \
         update-ca-certificates; \
       else \
         echo "No corporate CA supplied"; \
       fi \
    && rm -rf /var/lib/apt/lists/*

# Configure apt and install packages.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-utils \
        apt-transport-https \
        bash-completion \
        build-essential \
        ca-certificates \
        crudini \
        curl \
        dialog \
        fd-find \
        fonts-liberation \
        gconf-service \
        git \
        gnupg-agent \
        graphviz \
        groff \
        jq \
        less \
        libc-bin \
        libc6 \
        libcairo2 \
        libcups2 \
        libappindicator1 \
        libappindicator3-1 \
        libasound2 \
        libatk1.0-0 \
        libatk-bridge2.0-0 \
        libatspi2.0-0 \
        libbz2-dev \
        libdbus-1-3 \
        libexpat1 \
        libffi-dev \
        libfontconfig1 \
        libgbm1 \
        libgcc1 \
        libgconf-2-4 \
        libgdbm-dev \
        libgdk-pixbuf2.0-0 \
        libglib2.0-0 \
        libgtk-3-0 \
        libjq1 \
        liblzma-dev \
        libncurses5-dev \
        libnspr4 \
        libnss3 \
        libnss3-dev \
        libonig5 \
        libpango-1.0-0 \
        libpangocairo-1.0-0 \
        libreadline-dev \
        libsqlite3-dev \
        libssl-dev \
        libstdc++6 \
        libx11-6 \
        libx11-xcb1 \
        libxcb1 \
        libxcomposite1 \
        libxcursor1 \
        libxdamage1 \
        libxext6 \
        libxfixes3 \
        libxi6 \
        libxkbcommon0 \
        libxrandr2 \
        libxrender1 \
        libxss1 \
        libxtst6 \
        locales \
        lsb-release \
        nano \
        openssh-client \
        pipx \
        procps \
        python3-dev \
        python3-pip \
        python3-setuptools \
        rsync \
        software-properties-common \
        sudo \
        tk-dev \
        unzip \
        vim \
        wget \
        xdg-utils \
        xvfb \
        zip \
        zlib1g-dev

# Install AWS CLI and supporting Python packages.
RUN mkdir -p /tmp/aws \
    && cd /tmp/aws \
    && curl -fsSL \
         "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" \
         -o awscliv2.zip \
    && unzip awscliv2.zip \
    && ./aws/install \
    && pip3 install --break-system-packages \
         git-remote-codecommit \
         boto3 \
         tabulate \
    && if [ "$(dpkg --print-architecture)" = "arm64" ]; then \
         arch="arm64"; \
       else \
         arch="64bit"; \
       fi \
    && curl -fsSL \
         "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_${arch}/session-manager-plugin.deb" \
         -o /tmp/session-manager-plugin.deb \
    && dpkg -i /tmp/session-manager-plugin.deb \
    && rm -rf /tmp/aws /tmp/session-manager-plugin.deb

# Install LocalStack CLI globally.
RUN PIPX_HOME=/opt/pipx \
    PIPX_BIN_DIR=/usr/local/bin \
    pipx install --include-deps localstack \
    && PIPX_HOME=/opt/pipx \
       PIPX_BIN_DIR=/usr/local/bin \
       pipx install awscli-local

# Configure locales.
RUN echo "LC_ALL=en_US.UTF-8" >> /etc/environment \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && echo "LANG=en_US.UTF-8" > /etc/locale.conf \
    && locale-gen en_US.UTF-8

# Install Go.
RUN curl -fsSL \
      "https://golang.org/dl/go1.27.4.linux-$(dpkg --print-architecture).tar.gz" \
    | tar -xz -C /usr/local

ENV PATH="${PATH}:/usr/local/go/bin"

# Install GitHub CLI.
RUN mkdir -p -m 755 /etc/apt/keyrings \
    && wget -qO- \
         https://cli.github.com/packages/githubcli-archive-keyring.gpg \
         > /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo \
         "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
         > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/
