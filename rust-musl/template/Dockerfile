<%= generation_message %>
FROM instructure/core:latest
MAINTAINER Instructure

USER root

ENV RUST_VERSION <%= full_version %>
ENV RUST_DOWNLOAD_URL https://static.rust-lang.org/dist/rust-$RUST_VERSION-aarch64-unknown-linux-gnu.tar.gz
ENV RUST_DOWNLOAD_SHA256 <%= package_sha %>

ENV SSL_VERSION <%= ssl_version %>
ENV SSL_FLAVOR <%= ssl_flavor %>
ENV SSL_SHA <%= ssl_sha %>

RUN mkdir -p /usr/src/rust/

RUN curl -fsSL "$RUST_DOWNLOAD_URL" -o rust.tar.gz \
	&& echo "$RUST_DOWNLOAD_SHA256  rust.tar.gz" | sha256sum -c - \
	&& tar -C /usr/src/rust/ -xzf rust.tar.gz \
  && /usr/src/rust/rust-$RUST_VERSION-aarch64-unknown-linux-gnu/install.sh \
	&& rm rust.tar.gz

RUN apt-get update && \
	apt-get install -y \
		cmake \
		file \
		musl-tools \
		xutils-dev \
		&& \
	apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo -n "Downloading OpenSSL...\n" && \
    curl -O "https://www.openssl.org/source/old/$SSL_VERSION/openssl-$SSL_FLAVOR.tar.gz" && \
		echo "$SSL_SHA  openssl-$SSL_FLAVOR.tar.gz" && \
    tar -xvzf "openssl-$SSL_FLAVOR.tar.gz" && cd "openssl-$SSL_FLAVOR" && \
    echo -n "Configuring OpenSSL...\n" && \
    env CC=musl-gcc ./config --prefix=/usr/local/musl > /dev/null 2>&1 && \
    echo -n "Making OpenSSL...\n" && \
    env C_INCLUDE_PATH=/usr/local/musl/include/ make depend > /dev/null 2>&1 && \
    echo -n "Installing OpenSSL..\n." && \
    make > /dev/null 2>&1 && sudo make install > /dev/null 2>&1 && \
    echo -n "Removing OpenSS DirL..\n." && \
    cd .. && rm -rf "openssl-$SSL_FLAVOR.tar.gz" "openssl-$SSL_FLAVOR"

RUN mkdir -p /usr/src/app && chown docker:docker /usr/src/app && chown docker:docker /usr/src/rust
WORKDIR /usr/src/app

USER docker

ENV OPENSSL_DIR=/usr/local/musl/ \
    OPENSSL_INCLUDE_DIR=/usr/local/musl/include/ \
    DEP_OPENSSL_INCLUDE=/usr/local/musl/include/ \
    OPENSSL_LIB_DIR=/usr/local/musl/lib/ \
    OPENSSL_STATIC=1

ADD cargo-config.toml /home/docker/.cargo/config
