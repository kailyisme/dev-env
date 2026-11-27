FROM --platform=linux/amd64 debian:12.15

ENV DEBIAN_FRONTEND=noninteractive

ARG CORP_CA_B64=""

# Bootstrap the certificate store and add the optional corporate CA.
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates \
    && if [ -n "$CORP_CA_B64" ]; then \
         echo "$CORP_CA_B64" \
           | base64 -d \
           > /usr/local/share/ca-certificates/corp-ca.crt; \
         update-ca-certificates; \
       else \
         echo "No corporate CA supplied"; \
       fi \
    && rm -rf /var/lib/apt/lists/*

# Install system packages.
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
        libcomposite1 \
        libcursor1 \
        libdamage1 \
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

# Install AWS CLI and Session Manager plugin.
RUN mkdir -p /tmp/aws \
    && cd /tmp/aws \
    && curl -fsSL \
       "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" \
       -o awscliv2.zip \
    && unzip awscliv2.zip \
    && ./aws/install \
    && pip3 install --break-system-packages \
         boto3 \
         git-remote-codecommit \
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

# Install LocalStack CLI.
RUN pipx install --global --include-deps localstack \
    && pipx install --global awscli-local

# Configure locales.
RUN echo "LC_ALL=en_US.UTF-8" >> /etc/environment \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && echo "LANG=en_US.UTF-8" > /etc/locale.conf \
    && locale-gen en_US.UTF-8

# Install Go.
RUN GO_VERSION="1.27.4" \
    && architecture="$(dpkg --print-architecture)" \
    && curl -fsSL \
       "https://golang.org/dl/go${GO_VERSION}.linux-${architecture}.tar.gz" \
       | tar -xz -C /usr/local

ENV PATH="${PATH}:/usr/local/go/bin"

# Install GitHub CLI.
RUN mkdir -p -m 755 /etc/apt/keyrings \
    && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
       > /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo \
       "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
       > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*

# Install Poetry globally through pipx.
RUN pipx install --global poetry==2.5.1

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000

# Create the non-root development user.
RUN groupadd --gid "$USER_GID" "$USERNAME" \
    && useradd \
         --shell /bin/bash \
         --uid "$USER_UID" \
         --gid "$USER_GID" \
         --create-home \
         "$USERNAME"

# Configure passwordless sudo for runtime use.
RUN echo "$USERNAME ALL=(root) NOPASSWD:ALL" \
       > "/etc/sudoers.d/$USERNAME" \
    && chmod 0440 "/etc/sudoers.d/$USERNAME"

# Configure SSH for the development user.
RUN mkdir -p "/home/$USERNAME/.ssh" \
    && printf '%s\n' \
         "Host *" \
         "    IdentitiesOnly yes" \
         "    ServerAliveInterval 25" \
       > "/home/$USERNAME/.ssh/config" \
    && chmod 700 "/home/$USERNAME/.ssh" \
    && chmod 600 "/home/$USERNAME/.ssh/config" \
    && chown -R "$USERNAME:$USERNAME" "/home/$USERNAME/.ssh"

# Install the Starship binary system-wide.
RUN curl -fsSL https://starship.rs/install.sh \
    | sh -s -- --yes

# Install Terraform and Packer.
RUN wget -qO- https://apt.releases.hashicorp.com/gpg \
       | gpg --dearmor \
       > /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && chmod 644 /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && distribution="$(. /etc/os-release && echo "$VERSION_CODENAME")" \
    && echo \
       "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com ${distribution} main" \
       > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends terraform packer \
    && rm -rf /var/lib/apt/lists/*

# Install Hub.
RUN mkdir -p "/home/$USERNAME/.local/share" \
    && curl -fsSL \
       https://github.com/github/hub/releases/download/v2.14.2/hub-linux-amd64-2.14.2.tgz \
       -o /tmp/hub-linux-amd64-2.14.2.tgz \
    && tar -xzf /tmp/hub-linux-amd64-2.14.2.tgz \
       -C "/home/$USERNAME/.local/share" \
    && ln -s \
       "/home/$USERNAME/.local/share/hub-linux-amd64-2.14.2" \
       "/home/$USERNAME/.local/share/hub-linux" \
    && rm /tmp/hub-linux-amd64-2.14.2.tgz \
    && chown -R "$USERNAME:$USERNAME" "/home/$USERNAME/.local"

# Install yq.
RUN wget -q \
       https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 \
       -O /usr/local/bin/yq \
    && chmod 0755 /usr/local/bin/yq

# Install Chrome for Testing.
RUN CHROME_VERSION="$(curl -fsSL \
       https://googlechromelabs.github.io/chrome-for-testing/LATEST_RELEASE_STABLE)" \
    && wget -q \
       "https://storage.googleapis.com/chrome-for-testing-public/${CHROME_VERSION}/linux64/chrome-linux64.zip" \
       -O /tmp/chrome-linux64.zip \
    && unzip -q /tmp/chrome-linux64.zip -d /opt \
    && mv /opt/chrome-linux64 /opt/chrome \
    && ln -s /opt/chrome/chrome /usr/local/bin/chrome \
    && chmod 0755 /opt/chrome/chrome \
    && rm /tmp/chrome-linux64.zip

# Install ChromeDriver.
RUN CHROMEDRIVER_VERSION="$(curl -fsSL \
       https://googlechromelabs.github.io/chrome-for-testing/LATEST_RELEASE_STABLE)" \
    && wget -q \
       "https://storage.googleapis.com/chrome-for-testing-public/${CHROMEDRIVER_VERSION}/linux64/chromedriver-linux64.zip" \
       -O /tmp/chromedriver-linux64.zip \
    && unzip -q /tmp/chromedriver-linux64.zip -d /tmp \
    && install \
         -o root \
         -g root \
         -m 0755 \
         /tmp/chromedriver-linux64/chromedriver \
         /usr/local/bin/chromedriver \
    && rm -rf \
         /tmp/chromedriver-linux64 \
         /tmp/chromedriver-linux64.zip

# Install system-wide Python tooling.
RUN pip3 install --break-system-packages pre-commit

# Copy shell configuration after the user exists.
COPY --chown=vscode:vscode .bashrc /home/vscode/.bashrc
COPY --chown=vscode:vscode .bash_aliases /home/vscode/.bash_aliases

# Ensure the user owns directories used by its package managers.
RUN mkdir -p \
       "/home/$USERNAME/.local/bin" \
       "/home/$USERNAME/.local/share/java" \
    && chown -R "$USERNAME:$USERNAME" "/home/$USERNAME"

# Clean up system package data before the final user switch.
RUN apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Everything below this point is intentionally installed as vscode.
USER vscode
WORKDIR /home/vscode

SHELL ["/bin/bash", "-c"]

ENV NVM_DIR="/home/vscode/.nvm"
ENV PYENV_ROOT="/home/vscode/.pyenv"
ENV SDKMAN_DIR="/home/vscode/.sdkman"
ENV PATH="/home/vscode/.pyenv/bin:/home/vscode/.local/bin:${PATH}"

# Install NVM and Node.js.
RUN curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh \
       | bash \
    && source "$NVM_DIR/nvm.sh" \
    && nvm install 26.13.1 \
    && nvm alias default 26.13.1

# Install pyenv and Python.
RUN curl -fsSL https://pyenv.run | bash \
    && export PATH="$PYENV_ROOT/bin:$PATH" \
    && eval "$(pyenv init -)" \
    && pyenv install 3.13.18 \
    && pyenv global 3.13.18

# Install SDKMAN, Maven, Java, and WireMock.
RUN curl -fsSL https://get.sdkman.io | bash \
    && source "$SDKMAN_DIR/bin/sdkman-init.sh" \
    && sdk update \
    && sdk install maven 3.9.6 \
    && sdk install java 21.0.2-open \
    && mvn dependency:copy \
         -Dartifact=org.wiremock:wiremock-standalone:3.5.2 \
         -DoutputDirectory=/home/vscode/.local/share/java

ENV DEBIAN_FRONTEND=dialog

WORKDIR /workspaces
