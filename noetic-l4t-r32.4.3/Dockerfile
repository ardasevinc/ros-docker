ARG BASE_IMAGE=nvcr.io/nvidia/l4t-base:r32.4.3
FROM ${BASE_IMAGE} as l4t

ENV ROS_PYTHON_VERSION=3

ENV DEBIAN_FRONTEND=noninteractive

ENV TZ Europe/Istanbul
RUN ln -snf "/usr/share/zoneinfo/$TZ" /etc/localtime && echo "$TZ" > /etc/timezone

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        locales \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen en_US.UTF-8

# RUN apt-get update \
#     && apt-get install -y --no-install-recommends \
#     curl python3-pip ca-certificates python3-setuptools \
#     && apt-get clean \
#     && rm -rf /var/lib/apt/lists/*

# RUN pip3 install --no-cache-dir apt-mirror-updater \
#     && apt-mirror-updater -c http://ubuntu.vargonen.com/ubuntu/ \
#     && pip3 uninstall -y apt-mirror-updater


# add the ROS deb repo to the apt sources list
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
		cmake \
		build-essential \
		curl \
		wget \ 
		gnupg2 \
		lsb-release \
        ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
    
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get install -y --no-install-recommends \
        python3.10 \
        libqt5x11extras5-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install bootstrap dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
          python3-setuptools \
          python3-pip \
          libpython3-dev \
          python3-rosdep \
          python3-rosinstall-generator \
          python3-vcstool \
          build-essential && \
    rosdep init && \
    rosdep update && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir ros_catkin_ws \
    && cd ros_catkin_ws \
    && rosinstall_generator robot perception ros_canopen joy rviz --rosdistro noetic --deps --tar > noetic-ros_base.rosinstall \
    && mkdir src \
    && vcs import --input noetic-ros_base.rosinstall ./src \
    && apt-get update \
    && rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic --skip-keys python3-pykdl -y \
    && python3 ./src/catkin/bin/catkin_make_isolated --install --install-space \
    /opt/ros/noetic -DCMAKE_BUILD_TYPE=Release \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    && apt-get clean \ 
    && rm -rf /var/lib/apt/lists/*


# download/build the ROS source
# RUN mkdir ros_catkin_ws && \
#     cd ros_catkin_ws && \
#     rosinstall_generator ${ROS_PKG} vision_msgs --rosdistro ${ROS_DISTRO} --deps --tar > ${ROS_DISTRO}-${ROS_PKG}.rosinstall && \
#     mkdir src && \
#     vcs import --input ${ROS_DISTRO}-${ROS_PKG}.rosinstall ./src && \
#     apt-get update && \
#     rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro ${ROS_DISTRO} -y && \
#     python3 ./src/catkin/bin/catkin_make_isolated --install --install-space ${ROS_ROOT} -DCMAKE_BUILD_TYPE=Release && \
#     rm -rf /var/lib/apt/lists/*

# setup entrypoint
# COPY ./packages/ros_entrypoint.sh /ros_entrypoint.sh
# RUN echo 'source ${ROS_ROOT}/setup.bash' >> /root/.bashrc 
# ENTRYPOINT ["/ros_entrypoint.sh"]
# CMD ["bash"]
# WORKDIR /

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        tmux \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir /boldpilot
WORKDIR /boldpilot

COPY req.txt /tmp
RUN pip3 install --no-cache-dir -r /tmp/req.txt

RUN echo "source /opt/ros/noetic/setup.bash" >> /root/.bashrc

FROM l4t as zed

ARG L4T_MINOR_VERSION=32
ARG L4T_MINOR_VERSION=4
ARG ZED_SDK_MAJOR=3
ARG ZED_SDK_MINOR=7
ARG JETPACK_MAJOR=4
ARG JETPACK_MINOR=4

#This environment variable is needed to use the streaming features on Jetson inside a container
ENV LOGNAME root
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update -y && apt-get install --no-install-recommends lsb-release wget less udev sudo apt-transport-https build-essential cmake -y && \
    echo "# R32 (release), REVISION: ${L4T_MINOR_VERSION}" > /etc/nv_tegra_release ; \
    wget -q --no-check-certificate -O ZED_SDK_Linux_JP.run https://download.stereolabs.com/zedsdk/${ZED_SDK_MAJOR}.${ZED_SDK_MINOR}/jp${JETPACK_MAJOR}${JETPACK_MINOR}/jetsons && \
    chmod +x ZED_SDK_Linux_JP.run ; ./ZED_SDK_Linux_JP.run silent skip_tools && \
    rm -rf /usr/local/zed/resources/* \
    rm -rf ZED_SDK_Linux_JP.run && \
    rm -rf /var/lib/apt/lists/*

#This symbolic link is needed to use the streaming features on Jetson inside a container
RUN ln -sf /usr/lib/aarch64-linux-gnu/tegra/libv4l2.so.0 /usr/lib/aarch64-linux-gnu/libv4l2.so

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        mesa-utils \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /boldpilot