# Dockerfile to create IO500 container 
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Base deps + MPI + autotools needed by IOR/MDTest builds
RUN apt-get update && apt-get install -y --no-install-recommends \
    git iputils-ping net-tools dnsutils vim unzip curl \
    build-essential git ca-certificates \
    openmpi-bin libopenmpi-dev \
    numactl hwloc python3 \
    autoconf automake libtool pkg-config m4 \
    && rm -rf /var/lib/apt/lists/*

# Allow mpirun as root inside containers
ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

# ---- Get IO500 sources ----
WORKDIR /opt
ARG IO500_BRANCH=io500-isc24
RUN git clone https://github.com/IO500/io500.git -b ${IO500_BRANCH}
WORKDIR /opt/io500

# ---- Pre-seed pfind with a legacy LZ4 that provides lib/*.o ----
# pfind's compile.sh prefers ./lz4 if present; we pin to a commit with the old layout.
RUN mkdir -p /opt/io500/build && \
    cd /opt/io500/build && \
    git clone https://github.com/VI4IO/pfind.git && \
    cd pfind && \
    git clone https://github.com/lz4/lz4.git && \
    cd lz4 && \
    git checkout 5c434d38 && \
    make -j

# ---- Build the IO500 stack (IOR, MDTest, pfind) ----
RUN ./prepare.sh && make -j

# ---- SED tweaks ----
# 1) Make the wrapper script run mpirun itself (no recursion if you call io500.sh directly)
# 2) Use container-friendly defaults:
#    --allow-run-as-root (Docker), --oversubscribe (tiny test envs),
#    --bind-to none (avoid strict binding), and a small default of -np 2.
RUN sed -i 's|^io500_mpirun=.*|io500_mpirun=mpirun|' io500.sh && \
    sed -i 's|^io500_mpiargs=.*|io500_mpiargs="--allow-run-as-root --oversubscribe --bind-to none -np 2"|' io500.sh

# Provide a starter config that targets bind-mounted paths
RUN cp config-minimal.ini /opt/io500/myconfig.ini && \
    sed -i 's|^datadir =.*|datadir = /data|' /opt/io500/myconfig.ini && \
    sed -i 's|^resultdir =.*|resultdir = /results|' /opt/io500/myconfig.ini

# Convenience dirs (override with -v at runtime)
RUN mkdir -p /results /data
ENV IO500_WORKDIR=/data
ENV PATH="/opt/io500:${PATH}"

# Show help by default; override at runtime to run benchmarks
#CMD ["bash", "-lc", "echo 'Container ready. Try: ./io500.sh myconfig.ini' && ./io500 -h"]

