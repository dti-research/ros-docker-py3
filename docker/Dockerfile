FROM ubuntu:16.04

LABEL maintainer="Nicolai Anton Lynnerup nily@dti.dk"

ENV DEBIAN_FRONTEND="noninteractive" \
    TERM="xterm"

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

# Variables for ROS distribution, configuration, and relevant directories
ARG ROS_DISTRO="kinetic"
ARG ROS_CONFIG="desktop_full"

ENV CATKIN_WS="/usr/catkin_ws" \
    ROS_INSTALL_DIR="/opt/ros/$ROS_DISTRO" \
    ROS_PYTHON_VERSION=3\
    ROS_DISTRO=${ROS_DISTRO}

# Remove all py2 stuff
RUN apt-get remove python-*

# Install py3
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        python3 \
        python3-dev \
        python3-pip \
        python3-wheel \
        python3-setuptools \
        python3-empy

# Install ROS specific packages
RUN pip3 install \
        wheel \
        rosdep \
        rospkg \
        rosinstall_generator \
        rosinstall \
        wstool \
        vcstools \
        catkin_tools \
        catkin_pkg

# Initialise rosdep
RUN rosdep init && rosdep update

# Initialise Catkin workspace
RUN mkdir -p $CATKIN_WS/src $ROS_INSTALL_DIR

WORKDIR $CATKIN_WS

RUN rosinstall_generator $ROS_CONFIG --rosdistro $ROS_DISTRO \
    --deps --tar > .rosinstall \
    && wstool init -j$(nproc) src .rosinstall \
    && rosdep check --from-paths src --ignore-src | grep python | sed -e "s/^apt\t//g" | sed -z "s/\n/ /g" | sed -e "s/python/python3/g" \
    && rosdep install --from-paths src --ignore-src --rosdistro $ROS_DISTRO -y \
       --skip-keys="`rosdep check --from-paths src --ignore-src | grep python | sed -e "s/^apt\t//g" | sed -z "s/\n/ /g"`"

RUN find . -type f -exec sed -i 's/\/usr\/bin\/env[ ]*python/\/usr\/bin\/env python3/g' {} +

RUN catkin init \
    && catkin config --install --install-space $ROS_INSTALL_DIR \
       --cmake-args -DCMAKE_BUILD_TYPE=Release \
    && catkin build --no-status --no-summary --no-notify \
    && catkin clean -y --logs --build --devel

# Let ROS know where all of the system python3 packages are installed.
ENV PYTHONPATH=/usr/lib/python3/dist-packages

WORKDIR /
RUN rm -rf $CATKIN_WS

COPY ./entrypoint.sh .
ENTRYPOINT ["bash", "entrypoint.sh"]

CMD ["bash"]
