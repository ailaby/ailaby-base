ARG IMAGE_BASE="nvidia/cuda:11.8.0-base-ubuntu22.04"

# Caddy build
FROM ubuntu:22.04 as caddybuilder

ENV PATH=/opt/go/bin:$PATH
ENV GOROOT=/opt/go
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz && \
    tar -xf go1.22.4.linux-amd64.tar.gz && \
    mv ./go /opt && \ 
    wget https://github.com/caddyserver/xcaddy/releases/download/v0.4.2/xcaddy_0.4.2_linux_amd64.deb && \
    apt-get install ./xcaddy_0.4.2_linux_amd64.deb && \
    xcaddy build \
        --with github.com/caddyserver/replace-response && \
    mkdir -p /opt/caddy/bin && \
    mv ./caddy /opt/caddy/bin


# For build automation - Allows building from any Debian-based image

FROM ${IMAGE_BASE}

# System libs are handled in /etc/ld.so.conf.d
# Mamba environments only to use this variable
ENV LD_LIBRARY_PATH=

COPY --from=caddybuilder /opt/caddy/ /opt/caddy/

# Determines which scripts at /opt/ailaby/bin/build/layerX will be run by init.sh
ARG XPU_TARGET="GPU_NVIDIA" # GPU_AMD, CPU
ENV XPU_TARGET=${XPU_TARGET}

LABEL org.opencontainers.image.source https://github.com/ailaby/base

LABEL org.opencontainers.image.description "Base image for ailaby."

LABEL maintainer="voidump <voidump@gmail.com>"

SHELL ["/bin/bash", "-c"]
# Set ENV variables
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV TZ=UTC
ENV SHELL="/bin/bash"
ENV USER_NAME=user
ENV USER_PASSWORD=password
ENV USER_GROUPS=users,ailaby,adm,sudo,audio,video,tty,cdrom,dialout,dip,fax,floppy,lp,plugdev,ssl-cert,sudo,tape,voice
ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/opt/ailaby/bin:/opt/micromamba/bin:/opt/caddy/bin:$PATH
ENV OPT_SYNC=
ENV PYTHONUNBUFFERED=true
ENV APT_INSTALL="apt-get install -y --no-install-recommends"
ENV PIP_INSTALL="pip install --no-cache-dir"
ENV MAMBA_ROOT_PREFIX=/opt/micromamba
ENV MAMBA_EXE="/opt/micromamba/bin/micromamba"
ENV RCLONE_CONFIG="/etc/rclone/rclone.conf"
ENV IMAGE_SLUG="base"
# Copy early so we can use scripts in the build - Changes to these files will invalidate the cache and cause a rebuild.
COPY --chown=0:1111 ./COPY_ROOT/ /

ARG CUDA_STRING
ARG ROCM_STRING

# Use build scripts to ensure we can build all targets from one Dockerfile in a single layer.
# Don't put anything heavy in here - We can use multi-stage building above if necessary.
RUN yes | unminimize && \
    set -eo pipefail && /opt/ailaby/bin/build/layer0/init.sh | tee /var/log/build.log

# Copy overrides and new files into a final layer for fast rebuilds. Uncomment below
#COPY --chown=0:1111 ./COPY_ROOT_EXTRA/ /
#RUN set -eo pipefail && /opt/ailaby/bin/build/layer1/init.sh | tee -a /var/log/build.log

# Keep init.sh as-is and place additional logic in /opt/ailaby/bin/preflight.sh
CMD ["init.sh"]
