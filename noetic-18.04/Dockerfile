FROM docker.io/nvidia/cuda:11.7.0-devel-ubuntu18.04

ARG DEBIAN_FRONTEND=noninteractive

ENV TZ Europe/Istanbul
RUN ln -snf "/usr/share/zoneinfo/$TZ" /etc/localtime && echo "$TZ" > /etc/timezone

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        locales \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen en_US.UTF-8

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    curl python3-pip ca-certificates python3-setuptools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# RUN apt-key adv --fetch-keys \
# https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub \ 
#     && apt-get update \
#     && apt-get install -y --no-install-recommends \
#        curl python3-pip ca-certificates \
#     && apt-get clean \
#     && rm -rf /var/lib/apt/lists/*

RUN pip3 install --no-cache-dir apt-mirror-updater \
    && apt-mirror-updater -c http://ubuntu.vargonen.com/ubuntu/ \
    && pip3 uninstall -y apt-mirror-updater


#
# add the ROS deb repo to the apt sources list
#
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
		cmake \
		build-essential \
		wget \
		gnupg2 \
		lsb-release \
        python3 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -


#
# install bootstrap dependencies
#
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libpython3-dev \
        python3-rosdep \
        python3-rosinstall-generator \
        python3-vcstool \
        build-essential \
    && rosdep init \
    && rosdep update \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


#
# download/build the ROS source
#
RUN mkdir ros_catkin_ws \
    && cd ros_catkin_ws \
    && rosinstall_generator ros_base vision_msgs joy --rosdistro noetic --deps --tar > noetic-ros_base.rosinstall \
    && mkdir src \
    && vcs import --input noetic-ros_base.rosinstall ./src \
    && apt-get update \
    && rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic --skip-keys python3-pykdl -y \
    && python3 ./src/catkin/bin/catkin_make_isolated --install --install-space \
    /opt/ros/noetic -DCMAKE_BUILD_TYPE=Release \
    && apt-get clean \ 
    && rm -rf /var/lib/apt/lists/*

# Build ros joy


RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3 \
        tmux \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir /boldpilot
WORKDIR /boldpilot

# COPY py_requirements.txt /tmp
# RUN pip install --no-cache-dir -r /tmp/py_requirements.txt

RUN echo "source /opt/ros/noetic/setup.bash" >> /root/.bashrc