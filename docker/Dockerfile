FROM ubuntu:20.04
ARG DEBIAN_FRONTEND="noninteractive"

ARG WEST_VERSION=0.13.1
ARG CMAKE_VERSION=3.20.5
ARG NRFUTIL_VERSION=6.1.3
ARG NCS_VERSION=v1.8.0
ARG ZEPHYR_VERSION=v2.7.0-ncs1
ARG MCUBOOT_VERSION=v1.7.99-ncs4
ARG NRF_CMD_TOOL_URL="https://www.nordicsemi.com/-/media/Software-and-other-downloads/Desktop-software/nRF-command-line-tools/sw/Versions-10-x-x/10-15-4/nrf-command-line-tools-10.15.4_amd64.zip"

ARG WGET_ARGS="-q --show-progress --progress=bar:force:noscroll"

RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get install -y --no-install-recommends git wget ninja-build gperf \
    ccache dfu-util device-tree-compiler locales python3-dev python3-pip \
    python3-setuptools python3-tk python3-wheel udev xz-utils file make gcc \
    gcc-multilib g++-multilib libsdl2-dev unzip libncurses5 vim nano && \
    apt-get -y clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 install --no-cache-dir west==${WEST_VERSION} && \
    pip3 install --no-cache-dir cmake==${CMAKE_VERSION} && \
    pip3 install --no-cache-dir nrfutil==${NRFUTIL_VERSION}

# Download and unpack GNU Arm Embedded Toolchain (version 9-2019-q4-major)
RUN wget $WGET_ARGS -O archive.tar.bz2 "https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2?revision=108bd959-44bd-4619-9c19-26187abf5225&la=en&hash=E788CE92E5DFD64B2A8C246BBA91A249CB8E2D2D" \
    && echo fe0029de4f4ec43cf7008944e34ff8cc archive.tar.bz2 > /tmp/archive.md5 \
    && md5sum -c /tmp/archive.md5 \
    && rm /tmp/archive.md5 \
    && tar xf archive.tar.bz2 -C /opt \
    && rm archive.tar.bz2

# The following step replaces complete West workspace initialization just to install
# required Python dependencies from multiple Git repositories
ARG NRF_CONNECT_URL="https://raw.githubusercontent.com/nrfconnect"

RUN mkdir -p /tmp/nrf/scripts \
    && mkdir -p /tmp/zephyr/scripts \
    && mkdir -p /tmp/bootloader/mcuboot/scripts \
    && cd /tmp/nrf/scripts \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-nrf/${NCS_VERSION}/scripts/requirements.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-nrf/${NCS_VERSION}/scripts/requirements-base.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-nrf/${NCS_VERSION}/scripts/requirements-build.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-nrf/${NCS_VERSION}/scripts/requirements-doc.txt \
    && cd /tmp/zephyr/scripts \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-base.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-build-test.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-doc.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-run-test.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-extras.txt \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/requirements-compliance.txt \
    && cd /tmp/bootloader/mcuboot/scripts \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-mcuboot/${MCUBOOT_VERSION}/scripts/requirements.txt \
    && cd /tmp \
    && pip3 install --no-cache-dir -r zephyr/scripts/requirements.txt \
    && pip3 install --no-cache-dir -r nrf/scripts/requirements.txt \
    && pip3 install --no-cache-dir -r bootloader/mcuboot/scripts/requirements.txt \
    && rm -rf /tmp/*

# Nrf commandline tools
RUN mkdir nrf-commandline-tools && cd nrf-commandline-tools && \
    wget $WGET_ARGS "${NRF_CMD_TOOL_URL}" && \
    unzip nrf-command-line-tools-*.zip && \
    dpkg -i *.deb && \
    cd .. && rm -rf nrf-commandline-tools

# Set environment variables for Zephyr
ENV GNUARMEMB_TOOLCHAIN_PATH=/opt/gcc-arm-none-eabi-9-2019-q4-major
ENV ZEPHYR_TOOLCHAIN_VARIANT=gnuarmemb

# Set up locales
RUN locale-gen en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

ARG USER \
    host_uid
RUN useradd -l -m -s /bin/bash -u "$host_uid" $USER

USER $USER
WORKDIR /home/$USER

# Below actions are run with user context and permissions. They add West bash
# completions, colored output from GCC and saner tab completion behaviour
RUN cd ~ \
    && wget $WGET_ARGS ${NRF_CONNECT_URL}/sdk-zephyr/${ZEPHYR_VERSION}/scripts/west_commands/completion/west-completion.bash \
    && echo "source ~/west-completion.bash" >> ~/.bashrc \
    && echo 'export GCC_COLORS="error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01"' >> ~/.bashrc \
    && echo 'bind "TAB:menu-complete"' >> ~/.bashrc \
    && echo 'bind "set show-all-if-ambiguous on"' >> ~/.bashrc \
    && echo 'bind "set menu-complete-display-prefix on"' >> ~/.bashrc
