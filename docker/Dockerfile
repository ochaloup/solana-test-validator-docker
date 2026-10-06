FROM ubuntu:24.04@sha256:534baea6a22c03a63003dbc8dbe78fe34bc0d7e595d9a9dc9834884ff530eb55

ARG HOST_UID=1000
ARG HOST_GID=1000

ARG VERSION_SOLANA=3.1.12
ARG SHA256_SOLANA=4d45dab6ccfe5321292ad6dc152f7ace71ccbddde5ecda0aaa3f92bbfe8ed803
ARG VERSION_ANCHOR=1.2.1
ARG COMMIT_ANCHOR=f1d20a94e8857d66e39810039aa5b5eee9ff733e
ARG VERSION_RUST=1.97.1
ARG VERSION_RUSTUP=1.28.2
ARG SHA256_RUSTUP=20a06e644b0d9bd2fbdbfd52d42540bdde820ea7df86e92e533c073da0cdd43c
ARG VERSION_NODE=22.23.3
ARG SHA256_NODE=df450af89261115ef9f9e3830c3eeb2cc9213b63c720b1af623cb5dcbe2e02de

ENV TZ=Etc/UTC
ENV RUSTUP_HOME=/opt/.rust
ENV CARGO_HOME=/opt/.cargo
ENV NPM_CONFIG_PREFIX=/opt/.npm-global
ENV COREPACK_ENABLE_STRICT=0
ENV PATH="/opt/.cargo/bin:/opt/solana-release/bin:/opt/node/bin:/opt/.npm-global/bin:$PATH"

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      bash bzip2 ca-certificates curl xz-utils gcc-multilib build-essential autoconf automake libtool \
      zlib1g-dev git libssl-dev pkg-config procps libudev-dev llvm clang libclang-dev protobuf-compiler && \
    rm -rf /var/lib/apt/lists/*

# every download is checked against a hash pinned in this file, never one fetched next to it
RUN curl -fsSLo /tmp/node.tar.xz "https://nodejs.org/dist/v${VERSION_NODE}/node-v${VERSION_NODE}-linux-x64.tar.xz" && \
    echo "${SHA256_NODE}  /tmp/node.tar.xz" | sha256sum -c - && \
    mkdir -p /opt/node && \
    tar -xJf /tmp/node.tar.xz -C /opt/node --strip-components=1 && \
    rm /tmp/node.tar.xz && \
    corepack enable && \
    node --version

RUN curl -fsSLo /tmp/solana-release.tar.bz2 "https://github.com/anza-xyz/agave/releases/download/v${VERSION_SOLANA}/solana-release-x86_64-unknown-linux-gnu.tar.bz2" && \
    echo "${SHA256_SOLANA}  /tmp/solana-release.tar.bz2" | sha256sum -c - && \
    tar -xjf /tmp/solana-release.tar.bz2 -C /opt && \
    rm /tmp/solana-release.tar.bz2 && \
    solana --version

RUN groupadd -g $HOST_GID ubuntu || true && \
    useradd -m -s /bin/bash -u $HOST_UID -g $HOST_GID ubuntu || true && \
    mkdir -p /working_dir /opt/.npm-global /opt/.rust /opt/.cargo && \
    chown -R ubuntu:ubuntu /opt/solana-release /opt/.npm-global /opt/.rust /opt/.cargo /working_dir /home/ubuntu

USER ubuntu
WORKDIR /opt

RUN curl --proto '=https' --tlsv1.2 -fsSLo /tmp/rustup-init "https://static.rust-lang.org/rustup/archive/${VERSION_RUSTUP}/x86_64-unknown-linux-gnu/rustup-init" && \
    echo "${SHA256_RUSTUP}  /tmp/rustup-init" | sha256sum -c - && \
    chmod +x /tmp/rustup-init && \
    /tmp/rustup-init -y --no-modify-path --profile minimal --default-toolchain "${VERSION_RUST}" && \
    rm /tmp/rustup-init

RUN cargo install --git https://github.com/solana-foundation/anchor --rev "${COMMIT_ANCHOR}" anchor-cli --locked && \
    anchor --version | grep -q "${VERSION_ANCHOR}" && \
    rm -rf /opt/.cargo/registry /opt/.cargo/git

WORKDIR /working_dir

RUN solana config set --url localhost

# the keypair is created per container, never baked into a published layer
ENTRYPOINT ["/bin/bash", "-c", "test -f ~/.config/solana/id.json || solana-keygen new --no-bip39-passphrase -s -o ~/.config/solana/id.json; exec \"$@\"", "--"]
CMD ["/bin/bash"]
