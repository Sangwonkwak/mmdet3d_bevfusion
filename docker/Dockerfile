ARG PYTORCH="2.7.0"
ARG CUDA="12.8"
ARG CUDNN="9"

FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

# ENV TZ=Asia/Seoul \
ENV TORCH_CUDA_ARCH_LIST="8.6 9.0 12.0" \
    TORCH_NVCC_FLAGS="-Xfatbin -compress-all" \
    CMAKE_PREFIX_PATH="$(dirname $(which conda))/../" \
    FORCE_CUDA="1"

ENV DEBIAN_FRONTEND=noninteractive

# Avoid Public GPG key error
# https://github.com/NVIDIA/nvidia-docker/issues/1631
RUN rm /etc/apt/sources.list.d/cuda.list \
    && rm /etc/apt/sources.list.d/nvidia-ml.list \
    && apt-key del 7fa2af80 || true \
    && apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub \
    && apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/7fa2af80.pub

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ffmpeg \
        libsm6 \
        libxext6 \
        git \
        ninja-build \
        libglib2.0-0 \
        libxrender-dev \
        libxext6 \
        build-essential \
        cmake \
        nano \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# MMEngine, MMCV, MMDetection 설치 (OpenMMLab 2.0 스택)
RUN pip install --no-cache-dir -U openmim && \
    mim install "mmengine>=0.10.0,<1.0.0" && \
    mim install "mmcv==2.1.0" && \
    mim install "mmdet==3.3.0"

# MMDetection3D + BEVFusion 설치
RUN conda clean --all -y || true \
    && git clone https://github.com/Sangwonkwak/mmdet3d_bevfusion.git -b dev-1.x /mmdetection3d \
    && cd /mmdetection3d \
    # mmdet3d 자체를 editable 설치
    && pip install --no-cache-dir -e . \
    # BEVFusion 프로젝트 등록
    && python projects/BEVFusion/setup.py develop \
    && pip install --no-cache-dir nuscenes-devkit

ENV PYTHONPATH="/mmdetection3d:${PYTHONPATH}"

# WORKDIR /mmdetection3d

RUN echo "PS1='\\[\\e[1;33m\\](bevfusion)\\[\\e[0m\\] \\[\\e[1;31m\\]\\u\\[\\e[0m\\]@\\[\\e[1;36m\\]\\h\\[\\e[0m\\]:\\[\\e[1;34m\\]\\w\\[\\e[0m\\]\\# '" >> /root/.bashrc
