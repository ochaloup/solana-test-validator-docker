FROM ubuntu:24.04

# Accept UID as build argument
ARG HOST_UID=1000
ARG HOST_GID=1000

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC
ENV VERSION_SOLANA=2.3.7
ENV VERSION_ANCHOR=0.31.1
ENV VERSION_LIGHTPROTOCOL=0.27.0
ENV VERSION_PHOTON_INDEXER=49b7e7f0d668babbc4d65fe8a0a7236df76f75a8
ENV VERSION_YARN=4.1.1
ENV RUSTUP_HOME=/opt/.rust
ENV CARGO_HOME=/opt/.cargo
ENV NPM_CONFIG_PREFIX=/opt/.npm-global
ENV PATH="/opt/.cargo/bin:/opt/.npm-global/bin:$PATH"
ENV COREPACK_ENABLE_STRICT=0

# Update and install dependencies
RUN apt update && apt upgrade -y
RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC apt install -y bash bzip2 curl gcc-multilib build-essential autoconf automake libtool zlib1g-dev git libssl-dev pkg-config wget procps libudev-dev llvm libclang-dev protobuf-compiler

# Install Node.js directly from Ubuntu packages
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
    apt-get install -y nodejs

# Install yarn
RUN npm i -g yarn && \
    corepack enable && \
    yarn set version $VERSION_YARN

# Create ubuntu user with specific UID/GID matching host
RUN groupadd -g $HOST_GID ubuntu || true && \
    useradd -m -s /bin/bash -u $HOST_UID -g $HOST_GID ubuntu || true && \
    mkdir -p /working_dir /opt/.npm-global /opt/.rust /opt/.cargo && \
    chown -R ubuntu:ubuntu /opt /working_dir /home/ubuntu

# Set working directory
WORKDIR /opt

# Switch to ubuntu user for all installations
USER ubuntu
ENV PATH=/opt/.cargo/bin:/opt/.npm-global/bin:$PATH

# Install Rust (force rebuild this layer for version 1.89.0)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain 1.89.0 --profile minimal && \
    . /opt/.cargo/env

# Install Solana
RUN sh -c "$(curl -sSfL https://release.anza.xyz/v2.3.7/install)" && \
    echo 'export PATH="/home/ubuntu/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc && \
    echo 'export PATH="/home/ubuntu/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.profile && \
    export PATH="/home/ubuntu/.local/share/solana/install/active_release/bin:$PATH" && \
    solana --version

# Install light-protocol and anchor
RUN npm i -g @lightprotocol/zk-compression-cli@$VERSION_LIGHTPROTOCOL && \
    npm i -g @coral-xyz/anchor@$VERSION_ANCHOR

# Install photon indexer
RUN cargo install --git https://github.com/lightprotocol/photon.git --rev $VERSION_PHOTON_INDEXER --locked

# Install anchor directly (force rebuild this layer)
RUN cargo install --git https://github.com/coral-xyz/anchor anchor-cli --tag v$VERSION_ANCHOR --locked --force

# Update final PATH to include all necessary directories
ENV PATH="/opt/.cargo/bin:/home/ubuntu/.local/share/solana/install/active_release/bin:/opt/.npm-global/bin:$PATH"

# Prepare working directory
WORKDIR /working_dir

# Setup Solana configuration
RUN solana config set -ud && \
    solana-keygen new --no-bip39-passphrase -s -o ~/.config/solana/id.json && \
    solana config set --url localhost --keypair ~/.config/solana/id.json

# Set default command
CMD ["/bin/bash"]