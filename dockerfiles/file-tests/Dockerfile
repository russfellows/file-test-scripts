# Start with the Ubuntu 24.04 base image
FROM ubuntu:24.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# --- Add the local vdbench zip file to the image ---
# You must download vdbench50407.zip manually and place it
# in the same directory as this Dockerfile before building.
COPY vdbench50407.zip /tmp/vdbench.zip

# This single RUN layer performs all setup
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Install fio from the Ubuntu repository
    fio \
    # Install the Java 8 runtime required by vdbench
    openjdk-8-jre \
    # Install utilities needed for setup and general use
    unzip vim git curl \
    # Install networking tools
    iputils-ping net-tools dnsutils && \
    # Create the destination directory first
    mkdir /opt/vdbench && \
    # Unzip the archive directly into the new directory
    unzip /tmp/vdbench.zip -d /opt/vdbench/ && \
    # Clean up the temporary zip file and the apt cache
    rm /tmp/vdbench.zip && \
    rm -rf /var/lib/apt/lists/*


# Add the vdbench directory to the system PATH so you can run it from anywhere
ENV PATH="/opt/vdbench:${PATH}"

# Set a default working directory for when you enter the container
WORKDIR /data

# Set the default command to a shell for interactive use of the tools
CMD ["/bin/bash"]
