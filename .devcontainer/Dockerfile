FROM harbor.rnds.pro/dockerhub/library/debian:bullseye

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        autoconf \
        bison \
        bzip2 \
        libbz2-dev \
        ca-certificates \
        coreutils \
        libcurl4-openssl-dev \
        dpkg-dev \
        g++ \
        gcc \
        libgdbm-dev \
        git \
        libglib2.0-dev \
        libc6-dev \
        libffi-dev \
        libxml2-dev \
        libxslt-dev \
        linux-headers-generic \
        make \
        libncurses-dev \
        procps \
        libreadline-dev \
        tar \
        xz-utils \
        libyaml-dev \
        zlib1g-dev \
        dumb-init \
        fontconfig \
        libreoffice-writer \
        shared-mime-info \
        fonts-dejavu \
        fonts-droid-fallback \
        fonts-freefont-ttf \
        fonts-liberation \
        curl \
        gpg \
        sudo \
        wget \
        unzip \
        libpq-dev \
        postgresql-client \
        libsqlite3-dev \
        sqlite3 \
        rsync \
        openssh-client

RUN useradd -m -s /bin/bash dev && \
    echo "dev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/dev && \
    chmod 0440 /etc/sudoers.d/dev

USER dev
WORKDIR /home/dev

RUN install -dm 755 ~/.local/share/keyrings && \
    wget -qO - https://mise.jdx.dev/gpg-key.pub | gpg --dearmor | tee ~/.local/share/keyrings/mise-archive-keyring.gpg 1> /dev/null && \
    echo "deb [signed-by=$HOME/.local/share/keyrings/mise-archive-keyring.gpg arch=amd64] https://mise.jdx.dev/deb stable main" | sudo tee /etc/apt/sources.list.d/mise.list && \
    sudo apt-get update && \
    sudo apt-get install -y mise

RUN echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && \
echo 'eval "$(mise activate bash)"' >> ~/.bashrc

RUN mise use -g ruby@2.5.8 && \
    mise use -g node@14.17.0 && \
    mise install && \
    mise exec ruby -- gem install bundler -v 2.3.26 && \
    mise exec node -- npm install -g yarn@1.22.22

COPY --chown=dev:dev .devcontainer/library.zip /tmp/library.zip

RUN RUBY_PATH=$(mise exec ruby -- gem which rubygems | sed 's/\/lib\/ruby.*$//') && \
    SSL_DIRS=$(find $RUBY_PATH -name ssl_certs) && \
    for dir in $SSL_DIRS; do \
        echo "Установка сертификатов в $dir"; \
        sudo unzip -o /tmp/library.zip -d $dir; \
    done && \
    rm /tmp/library.zip

RUN echo 'registry=https://library.rnds.pro/repository/npm-proxy/' | sudo tee /etc/npmrc && \
    mise exec ruby -- bundle config set --global https://library.rnds.pro/repository/internal/ user:pass
