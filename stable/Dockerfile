FROM ubuntu:16.04 as builder

# set to --release for smaller, optimized library
ARG indy_build_flags=

# 3.5.x required for indy-node (von-network), 3.6.x for tob-api
ARG python_version=3.5.5

ARG indy_anoncreds_ver=1.0.32
ARG indy_crypto_ver=0.2.0
ARG indy_node_ver=1.2.297
ARG indy_plenum_ver=1.2.237
ARG python3_indy_ver=1.3.1-dev-408

ENV HOME=/home/indy

# Install environment
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        automake \
        bison \
        build-essential \
        ca-certificates \
        cmake \
        curl \
        flex \
        git \
        libbz2-dev \
        libffi-dev \
        libgmp-dev \
        libreadline-gplv2-dev \
        libsqlite3-dev \
        libsodium-dev \
        libssl-dev \
        libtool \
        libzmq3-dev \
        pkg-config && \
    rm -rf /var/lib/apt/lists/*

# Build and install pbc library
ARG pbc_lib_ver=0.5.14
RUN curl -o pbc-${pbc_lib_ver}.tar.gz https://crypto.stanford.edu/pbc/files/pbc-${pbc_lib_ver}.tar.gz && \
    tar xzvf pbc-${pbc_lib_ver}.tar.gz && \
    cd pbc-${pbc_lib_ver} && \
    ./configure && \
    make install && \
    cd .. && \
    rm -rf pbc-${pbc_lib_ver}*

ARG indy_crypto_rev=96c79b36c5056eade5a8e3bae418f5a733cc8d8d
WORKDIR $HOME

# Install rust toolchain and indy-sdk
RUN curl -o rustup https://sh.rustup.rs && \
    chmod +x rustup && \
    ./rustup -y && \
    git clone https://github.com/bcgov/indy-sdk.git && \
    git clone https://github.com/hyperledger/indy-crypto.git && \
    cd indy-sdk/libindy && \
    $HOME/.cargo/bin/cargo build ${indy_build_flags} && \
    mv target/*/libindy.so /usr/local/lib && \
    cd $HOME/indy-crypto/libindy-crypto && \
    git checkout ${indy_crypto_rev} && \
    $HOME/.cargo/bin/cargo build ${indy_build_flags} && \
    mv target/*/libindy_crypto.so /usr/local/lib && \
    cd $HOME && \
    rm -rf .cargo .multirust .profile .rustup rustup indy-sdk indy-crypto

# Install pyenv and python to user home directory
ENV PYENV_ROOT="$HOME/.pyenv"
ENV PATH="$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH"
RUN git clone https://github.com/pyenv/pyenv.git "$PYENV_ROOT" && \
    pyenv install $python_version && \
    pyenv global $python_version

# Update pip and install indy python libraries
RUN pip install --no-cache-dir --upgrade \
        pip~=9.0.0 \
        base58==0.2.5 \
        indy-anoncreds-dev==${indy_anoncreds_ver} \
        indy-crypto==${indy_crypto_ver} \
        indy-plenum-dev==${indy_plenum_ver} \
        indy-node-dev==${indy_node_ver} \
        python3-indy==${python3_indy_ver} \
        von-agent==0.6.5 \
        "git+https://github.com/PSPC-SPAC-buyandsell/didauth.git#master"


# start fresh
FROM ubuntu:16.04


ARG uid=1001

ENV HOME=/home/indy
ENV LC_ALL="C.UTF-8"
ENV LANG="C.UTF-8"
ENV SHELL=/bin/bash
ENV RUST_LOG=warning

RUN useradd -U -ms /bin/bash -u $uid indy

# Install environment
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        bzip2 \
        curl \
        git \
        libffi6 \
        libgmp10 \
        libreadline5 \
        libsodium18 \
        libzmq5 \
        openssl \
        sqlite3 && \
    rm -rf /var/lib/apt/lists/* /usr/share/doc/*

WORKDIR $HOME

COPY --from=builder /usr/local/lib/libpbc*.so /usr/local/lib/

COPY --from=builder /usr/local/lib/libindy*.so /usr/local/lib/

COPY --from=builder --chown=indy:indy $HOME .

ENV PYENV_ROOT="$HOME/.pyenv"
ENV PATH="$HOME/bin:$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH"

USER indy

CMD ["bash"]
