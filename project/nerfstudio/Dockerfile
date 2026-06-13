# syntax=docker/dockerfile:1
# Custom Dockerfile for nerfstudio on RTX 5060 Ti (Blackwell, sm_120)
# Base: CUDA 12.8.2 + PyTorch 2.7.1 + tiny-cuda-nn compiled for sm_120
ARG UBUNTU_VERSION=22.04
ARG NVIDIA_CUDA_VERSION=12.8.2
ARG CUDA_ARCHITECTURES=120

FROM nvidia/cuda:${NVIDIA_CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION} AS builder
ARG CUDA_ARCHITECTURES
ARG NVIDIA_CUDA_VERSION
ARG UBUNTU_VERSION

ENV DEBIAN_FRONTEND=noninteractive
ENV QT_XCB_GL_INTEGRATION=xcb_egl

RUN apt-get update && \
    apt-get install -y --no-install-recommends --no-install-suggests \
        git \
        wget \
        ninja-build \
        build-essential \
        libboost-program-options-dev \
        libboost-filesystem-dev \
        libboost-graph-dev \
        libboost-system-dev \
        libeigen3-dev \
        libflann-dev \
        libfreeimage-dev \
        libmetis-dev \
        libgoogle-glog-dev \
        libgtest-dev \
        libsqlite3-dev \
        libglew-dev \
        qtbase5-dev \
        libqt5opengl5-dev \
        libcgal-dev \
        libceres-dev \
        python3.10-dev \
        python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Install CMake 3.31
RUN wget https://github.com/Kitware/CMake/releases/download/v3.31.3/cmake-3.31.3-linux-x86_64.sh \
        -q -O /tmp/cmake-install.sh && \
    chmod u+x /tmp/cmake-install.sh && \
    mkdir /opt/cmake-3.31.3 && \
    /tmp/cmake-install.sh --skip-license --prefix=/opt/cmake-3.31.3 && \
    rm /tmp/cmake-install.sh && \
    ln -s /opt/cmake-3.31.3/bin/* /usr/local/bin

# Build and install GLOMAP
RUN git clone https://github.com/colmap/glomap.git && \
    cd glomap && \
    git checkout "1.0.0" && \
    mkdir build && cd build && \
    mkdir -p /build && \
    cmake .. -GNinja \
        "-DCMAKE_CUDA_ARCHITECTURES=${CUDA_ARCHITECTURES}" \
        -DCMAKE_INSTALL_PREFIX=/build/glomap && \
    ninja install -j$(nproc) && \
    cd ~ && rm -rf glomap

# Build and install COLMAP
RUN git clone https://github.com/colmap/colmap.git && \
    cd colmap && \
    git checkout "3.9.1" && \
    mkdir build && cd build && \
    cmake .. -GNinja \
        "-DCMAKE_CUDA_ARCHITECTURES=${CUDA_ARCHITECTURES}" \
        -DCMAKE_INSTALL_PREFIX=/build/colmap && \
    ninja install -j$(nproc) && \
    cd ~ && rm -rf colmap

# Install PyTorch 2.7.1 with CUDA 12.8 (first version with official Blackwell support)
RUN pip install --no-cache-dir --upgrade pip 'setuptools<70.0.0' && \
    pip install --no-cache-dir \
        torch==2.7.1+cu128 \
        torchvision==0.22.1+cu128 \
        'numpy<2.0.0' \
        --index-url https://download.pytorch.org/whl/cu128

# Install Hierarchical-Localization
RUN git clone --branch master --recursive \
        https://github.com/cvg/Hierarchical-Localization.git /opt/hloc && \
    cd /opt/hloc && git checkout v1.4 && \
    python3.10 -m pip install --no-cache-dir . && \
    cd ~

# Compile tiny-cuda-nn for Blackwell (sm_120)
RUN TCNN_CUDA_ARCHITECTURES=120 pip install --no-cache-dir \
    "git+https://github.com/NVlabs/tiny-cuda-nn.git#subdirectory=bindings/torch"

RUN pip install --no-cache-dir pycolmap==0.6.1 pyceres==2.1 omegaconf==2.3.0

# Build gsplat from source for sm_120, then install nerfstudio (main branch)
# nerfstudio main requires torch==2.7.1 and gsplat==1.4.0
RUN git clone --branch main --depth 1 \
        https://github.com/nerfstudio-project/nerfstudio.git /tmp/nerfstudio && \
    export TORCH_CUDA_ARCH_LIST="12.0" && \
    export MAX_JOBS=4 && \
    GSPLAT_VERSION=$(grep -oP 'gsplat==\K[^\s,"'"'"']+' /tmp/nerfstudio/pyproject.toml) && \
    pip install --no-cache-dir \
        git+https://github.com/nerfstudio-project/gsplat.git@v${GSPLAT_VERSION} && \
    pip install --no-cache-dir /tmp/nerfstudio 'numpy<2.0.0' && \
    rm -rf /tmp/nerfstudio

RUN chmod -R go=u /usr/local/lib/python3.10 && \
    chmod -R go=u /build

# ---------------------------------------------------------------------------
# Runtime stage
# ---------------------------------------------------------------------------
FROM nvidia/cuda:${NVIDIA_CUDA_VERSION}-runtime-ubuntu${UBUNTU_VERSION} AS runtime
ARG NVIDIA_CUDA_VERSION
ARG UBUNTU_VERSION

LABEL org.opencontainers.image.source="https://github.com/nerfstudio-project/nerfstudio"
LABEL org.opencontainers.image.licenses="Apache License 2.0"
LABEL org.opencontainers.image.base.name="docker.io/library/nvidia/cuda:${NVIDIA_CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION}"
LABEL org.opencontainers.image.documentation="https://docs.nerf.studio/"

RUN apt-get update && \
    apt-get install -y --no-install-recommends --no-install-suggests \
        libboost-filesystem1.74.0 \
        libboost-program-options1.74.0 \
        libc6 \
        libceres2 \
        libfreeimage3 \
        libgcc-s1 \
        libgl1 \
        libglew2.2 \
        libgoogle-glog0v5 \
        libqt5core5a \
        libqt5gui5 \
        libqt5widgets5 \
        python3.10 \
        python3.10-dev \
        python3-setuptools \
        build-essential \
        python-is-python3 \
        ffmpeg && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /build/colmap/ /usr/local/
COPY --from=builder /build/glomap/ /usr/local/
COPY --from=builder /usr/local/lib/python3.10/dist-packages/ /usr/local/lib/python3.10/dist-packages/
COPY --from=builder /usr/local/bin/ns* /usr/local/bin/

RUN /bin/bash -c 'ns-install-cli --mode install'

CMD /bin/bash -l
