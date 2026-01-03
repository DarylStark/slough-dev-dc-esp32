# hadolint ignore=DL3007
FROM dast1986/slough-dev-dc-cpp:1.0.0 AS base

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Switch to root user
USER root

# Install required packages
# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    wget \
    flex \
    bison \
    gperf \
    python3 \
    python3-pip \
    python3-venv \
    cmake \
    ninja-build \
    ccache \
    libffi-dev \
    libssl-dev \
    dfu-util \
    libusb-1.0-0 \
    screen \
    psmisc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Switch back to developer user
USER developer

# Install ESP32 IDF
WORKDIR /home/developer
RUN mkdir esp
WORKDIR /home/developer/esp
RUN git clone --branch v5.5.2 --recursive https://github.com/espressif/esp-idf.git --depth 1 --recurse-submodules --shallow-submodules
WORKDIR /home/developer/esp/esp-idf
RUN ./install.sh all

# Copy the ESP32 IDF activation
WORKDIR /home/developer
COPY bashrc.local .bashrc.local

# Configure Starship
ENV PROMPTNAME="ESP32"

# Set the default command to bash
CMD ["bash"]
