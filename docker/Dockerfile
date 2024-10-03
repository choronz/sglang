ARG CUDA_VERSION=12.4.1
FROM nvidia/cuda:${CUDA_VERSION}-devel-ubuntu22.04
ARG BUILD_TYPE=all
ARG PYTHON_VERSION=3.12
ENV DEBIAN_FRONTEND=noninteractive

# Install Python and other dependencies
RUN echo 'tzdata tzdata/Areas select America' | debconf-set-selections \
    && echo 'tzdata tzdata/Zones/America select Los_Angeles' | debconf-set-selections \
    && apt-get update -y \
    && apt-get install -y ccache software-properties-common git curl sudo libibverbs-dev \
    && add-apt-repository ppa:deadsnakes/ppa \
    && apt-get update -y \
    && apt-get install -y python${PYTHON_VERSION} python${PYTHON_VERSION}-dev \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 1 \
    && update-alternatives --set python3 /usr/bin/python${PYTHON_VERSION} \
    && ln -sf /usr/bin/python${PYTHON_VERSION}-config /usr/bin/python3-config \
    && curl -sS https://bootstrap.pypa.io/get-pip.py | python${PYTHON_VERSION} \
    && python3 --version && python3 -m pip --version \
    && rm -rf /var/lib/apt/lists/* && apt clean
    
# RUN echo 'tzdata tzdata/Areas select America' | debconf-set-selections \
#     && echo 'tzdata tzdata/Zones/America select Los_Angeles' | debconf-set-selections \
#     && apt update -y \
#     && apt install software-properties-common -y \
#     && add-apt-repository ppa:deadsnakes/ppa -y && apt update \
#     && apt install python3.12 python3.12-dev -y \
#     && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1 && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 2 \
#     && update-alternatives --set python3 /usr/bin/python3.12 && apt install python3.12-distutils -y \
#     && apt install curl git sudo libibverbs-dev -y \
#     && curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py \
#     && python3 --version && python3 -m pip --version \
#     && rm -rf /var/lib/apt/lists/* \
#     && apt clean

# For openbmb/MiniCPM models
RUN pip3 install datamodel_code_generator

WORKDIR /sgl-workspace

RUN python3 -m pip install --upgrade pip setuptools wheel html5lib six \
    && git clone --depth=1 https://github.com/sgl-project/sglang.git \
    && cd sglang \
    && if [ "$BUILD_TYPE" = "srt" ]; then \
         python3 -m pip --no-cache-dir install -e "python[srt]"; \
       else \
         python3 -m pip --no-cache-dir install -e "python[all]"; \
       fi

ARG CUDA_VERSION
RUN if [ "$CUDA_VERSION" = "12.1.1" ]; then \
        export CUDA_IDENTIFIER=cu121 && \
        python3 -m pip --no-cache-dir install flashinfer -i https://flashinfer.ai/whl/cu121/torch2.4/; \
    elif [ "$CUDA_VERSION" = "12.4.1" ]; then \
        export CUDA_IDENTIFIER=cu124 && \
        python3 -m pip --no-cache-dir install flashinfer -i https://flashinfer.ai/whl/cu124/torch2.4/; \
    else \
        echo "Unsupported CUDA version: $CUDA_VERSION" && exit 1; \
    fi

RUN python3 -m pip cache purge


ENV DEBIAN_FRONTEND=interactive
