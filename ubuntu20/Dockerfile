FROM nvidia/opengl:1.0-glvnd-devel-ubuntu20.04 as glvnd

FROM nvidia/cuda:11.4.1-cudnn8-devel-ubuntu20.04

ARG CONTAINER=ubuntu20
# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt update && \
    apt install -q -y --no-install-recommends tzdata

RUN apt-get update && apt-get install -q -y --no-install-recommends \
    dirmngr \
    gnupg2 lsb-release

# setup sources.list
RUN echo "deb http://packages.ros.org/ros2/ubuntu focal main" > /etc/apt/sources.list.d/ros2-latest.list

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

#setup environment 
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO foxy

#install ros2 packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-ros-core

#install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    build-essential \
    git \
    python3-colcon-common-extensions \
    python3-colcon-mixin \
    python3-rosdep \
    python3-vcstool

# bootstrap rosdep
RUN rosdep init && \
    rosdep update --rosdistro $ROS_DISTRO

# setup colcon mixin and metadata
RUN colcon mixin add default \
      https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml && \
    colcon mixin update && \
    colcon metadata add default \
      https://raw.githubusercontent.com/colcon/colcon-metadata-repository/master/index.yaml && \
    colcon metadata update

# install ros2 packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-ros-base

ENV ROS2_DISTRO foxy

# install ros2 packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-demo-nodes-cpp \
    ros-foxy-demo-nodes-py
    

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-desktop

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D2486D2DD83DB69272AFE98867170598AF249743

# setup sources.list
RUN . /etc/os-release \
    && echo "deb http://packages.osrfoundation.org/gazebo/$ID-stable `lsb_release -sc` main" > /etc/apt/sources.list.d/gazebo-latest.list

# install gazebo packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    gazebo11
    
# install gazebo packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgazebo11-dev

#####################################
RUN apt-get update && apt-get install -y xauth && apt-get upgrade -y
# setup environment

RUN apt-get update && apt-get install -y \
    libgl1-mesa-glx \
    libgl1-mesa-dri

RUN apt-get update && apt-get install -y --no-install-recommends \
    libglvnd0 \
    libgl1 \
    libglx0 \
    libegl1 \
    libgles2

COPY --from=glvnd /usr/share/glvnd/egl_vendor.d/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}

ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

RUN if ! command -v sudo >/dev/null; then       apt-get update       && apt-get install -y sudo       && apt-get clean;     fi

RUN apt-get update && apt-get install -y openssh-server iproute2
RUN mkdir /var/run/sshd

RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


RUN useradd -rm -p '' -d /home/ros -s /bin/bash -g root -G sudo -u 1001 ros  &&  echo "ros ALL=NOPASSWD: ALL" >> \
/etc/sudoers.d/rocker &&  sudo usermod -a -G video ros && mkdir -p /root/.ssh/ /home/ros/.ssh/ 

COPY authorized_keys /root/.ssh/authorized_keys
COPY authorized_keys /home/ros/.ssh/authorized_keys
COPY ${CONTAINER}/_init.sh /_init.sh
RUN chown -hR ros /home/ros/.ssh && ssh-keygen -A && chmod 755 /_init.sh

RUN echo "ros2init () { source /opt/ros/foxy/setup.bash; }\n \
source /usr/share/gazebo/setup.sh\nif [ -f /.env ]\nthen\n  \
export \$(cat /.env | sed 's/#.*//g' | xargs)\nfi\n\
export CUDA_HOME=/usr/local/cuda\n\
export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64\n\
export PATH=\$PATH:\$CUDA_HOME/bin" >> /etc/bash.bashrc

SHELL [ "/bin/bash", "-c" ]
CMD ["sudo","/_init.sh","2022"]