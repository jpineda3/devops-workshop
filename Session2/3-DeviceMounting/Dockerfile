FROM ubuntu:22.04

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install useful tools for serial device testing
RUN apt-get update && apt-get install -y \
    minicom \
    screen \
    picocom \
    cu \
    python3 \
    python3-pip \
    python3-serial \
    usbutils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

CMD ["/bin/bash"]
