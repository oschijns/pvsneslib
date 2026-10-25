# syntax=docker/dockerfile:1
ARG BASE_IMAGE=retrodev:latest

# MARK: STAGE 1
# Stage 1 - Build the toolchain
FROM ubuntu:24.04 AS builder
ARG JOBS=1

# Set the default username for the container
ARG USERNAME=retrodev

# Setup paths for the pvsneslib toolchain
ARG SNES_INST=/pvsneslib

# Switch to root to setup the environment
USER root

# Setup paths for the pvsneslib toolchain
ENV      SNES_INST=${SNES_INST}
ENV PVSNESLIB_HOME=/tmp/pvsneslib

# Install system dependencies
RUN apt update \
    && apt upgrade -y \
    && apt install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        cmake           \
        make            \
        gcc             \
        g++             \
        gcc-12          \
        git             \
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 90 \
    && apt autoremove -yq \
    && apt clean          \
    && rm -rf /var/lib/apt/lists/*

# Build the toolchain and cleanup unnecessary files
RUN git clone --branch master --depth 1 --recurse-submodules "https://github.com/alekmaul/pvsneslib.git" ${PVSNESLIB_HOME} \
    && cd ${PVSNESLIB_HOME} \
    && make -j"${JOBS}" pvsneslib \
    && make -j"${JOBS}" tools     \
    && mkdir -p ${SNES_INST}/devkitsnes/ ${SNES_INST}/pvsneslib/ \
    && cp -r  /tmp/pvsneslib/devkitsnes/bin     ${SNES_INST}/devkitsnes/bin     \
    && cp -r  /tmp/pvsneslib/devkitsnes/include ${SNES_INST}/devkitsnes/include \
    && cp -r  /tmp/pvsneslib/devkitsnes/tools   ${SNES_INST}/devkitsnes/tools   \
    && cp -r  /tmp/pvsneslib/pvsneslib/include  ${SNES_INST}/pvsneslib/include  \
    && cp -r  /tmp/pvsneslib/pvsneslib/lib      ${SNES_INST}/pvsneslib/lib      \
    && cp -r  /tmp/pvsneslib/pvsneslib/res      ${SNES_INST}/pvsneslib/res      \
    && cp -r  /tmp/pvsneslib/pvsneslib/snesmod  ${SNES_INST}/pvsneslib/snesmod  \
    && cp -r  /tmp/pvsneslib/pvsneslib/source   ${SNES_INST}/pvsneslib/source   \
    && rm -rf /tmp/pvsneslib


# MARK: STAGE 2
# Stage 2 - Prepare minimal image
FROM ubuntu:24.04 AS runner

# Set the default username for the container
ARG USERNAME=pvsneslib

# Setup paths for the pvsneslib toolchain
ARG SNES_INST=/pvsneslib

# Switch to root to setup the environment
USER root

# Install packages to work on C/C++ projects, 
# including Python, Git
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        sudo            \
        pkg-config      \
        build-essential \
        gcc             \
        clang           \
        make            \
        cmake           \
        xmake           \
        ninja-build     \
        xxd             \
        git             \
        curl            \
        wget            \
        zip             \
        unzip           \
    && useradd --create-home --shell /bin/bash ${USERNAME} \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME} \
    && apt-get autoremove -yq \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Setup paths for the pvsneslib toolchain
ENV      SNES_INST=${SNES_INST}
ENV PVSNESLIB_HOME=${SNES_INST}
ENV           PATH="${SNES_INST}/bin:$PATH"

# Copy over built toolchain from previous stage
COPY --from=builder ${SNES_INST} ${SNES_INST}

    
# Switch back to runner user
USER ${USERNAME}
WORKDIR /home/${USERNAME}