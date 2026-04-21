### This is a full chipyard setup

# BUILD BASE FOR CI

FROM ubuntu:24.04 AS base
ARG CHIPYARD_HASH
ARG CHIPYARD_REPO=https://github.com/ucb-bar/chipyard.git

MAINTAINER https://groups.google.com/forum/#!forum/chipyard

SHELL ["/bin/bash", "-c"]

# Install dependencies needed by Chipyard setup scripts
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    curl \
    wget \
    git \
    sudo \
    ca-certificates \
    keyboard-configuration \
    console-setup \
    bc \
    unzip

WORKDIR /root

# Install Chipyard and run ubuntu-req.sh to install necessary dependencies
RUN git clone $CHIPYARD_REPO chipyard && \
        cd chipyard && \
        git checkout $CHIPYARD_HASH

RUN ./chipyard/.github/scripts/install-conda.sh

# BUILD IMAGE WITH TOOLCHAINS

# Use above build as base
FROM base AS base-with-tools

SHELL ["/bin/bash", "-cl"]

# Initialize repo
RUN cd chipyard && \
        export MAKEFLAGS="-j$(nproc)" && \
        source /opt/conda/etc/profile.d/conda.sh && \
        conda activate base && \
        ./build-setup.sh --conda-env-name chipyard

SHELL ["/opt/conda/bin/conda", "run", "-n", "chipyard", "/bin/bash", "-cl"]

# Set up FireMarshal. Building and cleaning br-base.json builds the underlying
# buildroot image (which takes a long time) but doesn't keep all the br-base
# stuff around (since that's faster to rebuild).
RUN cd chipyard && \
        source env.sh && \
        cd software/firemarshal && \
        ./init-submodules.sh && \
        pip3 install -r python-requirements.txt && \
        ./marshal build br-base.json && \
        ./marshal clean br-base.json

# Run script to set environment variables on entry
ENTRYPOINT ["chipyard/scripts/entrypoint.sh"]

# END IMAGE CUSTOMIZATIONS

CMD ["/bin/sh"]
