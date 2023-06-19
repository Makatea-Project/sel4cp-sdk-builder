##
## Docker image ready to build the sel4cp SDK for all supported
## targets and boards.
##

##
## ARGs used in multiple images must be defined before the first FROM.
##
ARG ARM_TOOLCHAIN_TARGET=aarch64-none-elf
ARG RISCV_TOOLCHAIN_TARGET=riscv64-unknown-elf

##
## Build and install musl to /opt/musl.
##
FROM debian:stable AS musl
ARG MUSL_VERSION=1.2.4
ARG MUSL_BASEURL=https://musl.libc.org/releases
ARG MUSL_BASENAME=musl-$MUSL_VERSION
ARG MUSL_URL=$MUSL_BASEURL/$MUSL_BASENAME.tar.gz
RUN set -x                                                              \
    && apt-get -y update                                                \
    && apt-get -y install --no-install-recommends                       \
       ca-certificates                                                  \
       curl                                                             \
       gcc                                                              \
       make                                                             \
       xz-utils                                                         \
    && curl "$MUSL_URL" | tar -C /tmp -xzf-                             \
    && cd /tmp/$MUSL_BASENAME                                           \
    && ./configure --prefix=/opt/musl --enable-wrapper=gcc              \
    && make -j $(nproc)                                                 \
    && make install

##
## Unpack the ARM cross toolchain to /opt/toolchain/$ARM_TOOLCHAIN_TARGET
##
FROM debian:stable AS arm-toolchain
ARG ARM_TOOLCHAIN_TARGET
ARG ARM_TOOLCHAIN_VERSION=10.3-2021.07
ARG ARM_TOOLCHAIN_BASEURL=https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-a/$ARM_TOOLCHAIN_VERSION/binrel
ARG ARM_TOOLCHAIN_BASENAME=gcc-arm-$ARM_TOOLCHAIN_VERSION-x86_64-$ARM_TOOLCHAIN_TARGET
ARG ARM_TOOLCHAIN_URL=$ARM_TOOLCHAIN_BASEURL/$ARM_TOOLCHAIN_BASENAME.tar.xz
RUN set -x                                                              \
    && apt-get -y update                                                \
    && apt-get -y install --no-install-recommends                       \
       ca-certificates                                                  \
       curl                                                             \
       xz-utils                                                         \
    && mkdir -p /opt/toolchain/"$ARM_TOOLCHAIN_TARGET"                  \
    && cd /opt/toolchain/"$ARM_TOOLCHAIN_TARGET"                        \
    && curl "$ARM_TOOLCHAIN_URL" | tar -xJf- --strip-components=1       \
    && chown -R 0:0 .

##
## Unpack the RISC-V cross toolchain to /opt/toolchain/$RISCV_TOOLCHAIN_TARGET
##
FROM debian:stable AS riscv-toolchain
ARG RISCV_TOOLCHAIN_TARGET
ARG RISCV_TOOLCHAIN_RELEASE=2020.12
ARG RISCV_TOOLCHAIN_VERSION=10.2.0-2020.12.8
ARG RISCV_TOOLCHAIN_BASEURL=https://static.dev.sifive.com/dev-tools/freedom-tools/v$RISCV_TOOLCHAIN_RELEASE
ARG RISCV_TOOLCHAIN_BASENAME=riscv64-unknown-elf-toolchain-$RISCV_TOOLCHAIN_VERSION-x86_64-linux-ubuntu14
ARG RISCV_TOOLCHAIN_URL=$RISCV_TOOLCHAIN_BASEURL/$RISCV_TOOLCHAIN_BASENAME.tar.gz
RUN set -x                                                              \
    && apt-get -y update                                                \
    && apt-get -y install --no-install-recommends                       \
       ca-certificates                                                  \
       curl                                                             \
    && mkdir -p /opt/toolchain/"$RISCV_TOOLCHAIN_TARGET"                \
    && cd /opt/toolchain/"$RISCV_TOOLCHAIN_TARGET"                      \
    && curl "$RISCV_TOOLCHAIN_URL" | tar -xzf- --strip-components=1     \
    && chown -R 0:0 .

##
## Build the main image.
##
FROM debian:stable

# Install Debian packages.
RUN set -x                                                              \
    && apt-get -y update                                                \
    && apt-get -y install --no-install-recommends                       \
       cmake                                                            \
       device-tree-compiler                                             \
       gcc                                                              \
       git                                                              \
       ipxe-qemu                                                        \
       libc6-dev                                                        \
       libxml2-utils                                                    \
       make                                                             \
       musl                                                             \
       ninja-build                                                      \
       pandoc                                                           \
       python3                                                          \
       python3-venv                                                     \
       qemu-system-arm                                                  \
       qemu-system-misc                                                 \
       texlive-fonts-recommended                                        \
       texlive-latex-extra                                              \
       texlive-latex-recommended                                        \
       xz-utils                                                         \
    && apt-get clean                                                    \
    && rm -rf /var/cache/apt

# Install the python packages.
COPY requirements.txt /tmp/
RUN set -x                                                              \
    && python3 -m venv /opt/pyenv                                       \
    && /opt/pyenv/bin/pip install --upgrade                             \
         future                                                         \
         jsonschema                                                     \
         setuptools                                                     \
         six                                                            \
         wheel                                                          \
         pip                                                            \
    && /opt/pyenv/bin/pip install -r /tmp/requirements.txt              \
    && rm -f /tmp/requirements.txt                                      \
    && PATH=/opt/pyenv/bin:$PATH

# Copy the musl and toolchain files.
COPY --from=musl /opt/musl/ /opt/musl/
COPY --from=arm-toolchain /opt/toolchain/ /opt/toolchain/
COPY --from=riscv-toolchain /opt/toolchain/ /opt/toolchain/

# Add the tools to the PATH.
ARG ARM_TOOLCHAIN_TARGET
ARG RISCV_TOOLCHAIN_TARGET
ENV PATH=/opt/pyenv/bin:/opt/musl/bin:/opt/toolchain/$ARM_TOOLCHAIN_TARGET/bin:/opt/toolchain/$RISCV_TOOLCHAIN_TARGET/bin:$PATH
