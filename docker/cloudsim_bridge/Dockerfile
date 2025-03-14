# Ubuntu 18.04 with nvidia-docker2 beta opengl support
#FROM osrf/subt-virtual-testbed:models_latest
FROM nvidia/opengl:1.0-glvnd-devel-ubuntu18.04

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get update \
 && apt-get install --no-install-recommends -y \
    tzdata \
    sudo \
    wget \
    gnupg \
    lsb-release \
 && ln -fs /usr/share/zoneinfo/America/Los_Angeles /etc/localtime \
 && dpkg-reconfigure --frontend noninteractive tzdata \
 && apt-get clean

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`
ARG user_id
ENV USERNAME developer
RUN useradd -U --uid ${user_id} -ms /bin/bash $USERNAME \
 && echo "$USERNAME:$USERNAME" | chpasswd \
 && adduser $USERNAME sudo \
 && echo "$USERNAME ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USERNAME

# Commands below run as the developer user
USER $USERNAME

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME

# Tools I find useful during development
RUN sudo apt-get update -qq \
 && sudo apt-get install --no-install-recommends -y -qq \
        build-essential \
        bwm-ng \
        atop \
        cmake \
        cppcheck \
        expect \
        gdb \
        git \
        gnutls-bin \
        libbluetooth-dev \
        libccd-dev \
        libcwiid-dev \
        libfcl-dev \
        libgoogle-glog-dev \
        libspnav-dev \
        libusb-dev \
        lsb-release \
        python3-dbg \
        python3-empy \
        python3-numpy \
        python3-setuptools \
        python3-pip \
        python3-venv \
        ruby2.5 \
        ruby2.5-dev \
        software-properties-common \
        sudo \
        tcl8.6 \
        vim \
        wget \
        net-tools \
        iputils-ping \
        g++-8 \
 && sudo apt-get clean -qq

RUN sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8 --slave /usr/bin/gcov gcov /usr/bin/gcov-8

RUN gcc --version

# Install AWS CLI. This is needed by cloudsim to capture ROS logs.
RUN pip3 install --upgrade awscli=="1.16.220"
VOLUME /root/.aws

# install ROS and required packages
RUN sudo /bin/sh -c 'echo "deb [trusted=yes] http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' \
 && sudo apt-get update \
 && sudo apt-get install -y \
    libpcl-dev \
    libpcl-conversions-dev \
    python-catkin-tools \
    python-rosdep \
    python-rosinstall \
    ros-melodic-desktop \
    ros-melodic-joystick-drivers \
    ros-melodic-pcl-ros \
    ros-melodic-pointcloud-to-laserscan \
    ros-melodic-robot-localization \
    ros-melodic-spacenav-node \
    ros-melodic-tf2-sensor-msgs \
    ros-melodic-twist-mux \
    ros-melodic-rviz-imu-plugin \
    ros-melodic-rotors-control \
    ros-melodic-theora-image-transport \
 && sudo rosdep init \
 && sudo apt-get clean

# sdformat8-sdf conflicts with sdformat-sdf installed from gazebo
# so we need to workaround this using a force overwrite
# Do this before installing ign-gazebo
RUN sudo /bin/sh -c 'echo "deb [trusted=yes] http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list' \
 && sudo /bin/sh -c 'wget http://packages.osrfoundation.org/gazebo.key -O - | apt-key add -' \
 && sudo /bin/sh -c 'apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654'

# install ign-dome and the ign-bridge
RUN sudo apt-get update \
&&  sudo apt-get install -y \
    ignition-dome \
 && sudo apt-get clean

# install the ros to ign bridge
RUN sudo apt-get update \
 && sudo apt-get install -y \
    ros-melodic-ros-ign \
 && sudo apt-get clean -qq

# Make a couple folders for organizing docker volumes
RUN mkdir ~/workspaces ~/other

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME

# Clone all the subt models so that you don't download them every time
# docker is run
RUN mkdir -p subt_ws/src \
 && cd subt_ws/src \
 && git config --global http.postBuffer 1048576000 \
 && git clone https://github.com/osrf/subt

# Install dependencies
RUN . /opt/ros/melodic/setup.sh \
 && rosdep update \
 && sudo apt-get update \
 && rosdep install -y --from-paths . --ignore-src -r \
    --skip-keys=ignition-transport9 \
    --skip-keys=ignition-math6 \
    --skip-keys=ignition-msgs6 \
    --skip-keys=ignition-plugin1 \
    --skip-keys=ignition-common3 \
    --skip-keys=ignition-launch2 \
 && sudo apt-get clean -qq

WORKDIR /home/$USERNAME/subt_ws

RUN /bin/bash -c 'source /opt/ros/melodic/setup.bash && catkin_make -DCMAKE_BUILD_TYPE=Release install'
RUN /bin/sh -c 'echo ". /opt/ros/melodic/setup.bash" >> ~/.bashrc' \
 && /bin/sh -c 'echo ". ~/subt_ws/install/setup.sh" >> ~/.bashrc'

# Create a directory required by cloudsim
RUN mkdir -p /tmp/ign

# Copy entry point script, and set the entrypoint
COPY cloudsim_bridge/run_bridge.bash ./
ENTRYPOINT ["./run_bridge.bash"]

# Customize your image here.
# E.g.:
# ENV PATH="/opt/sublime_text:$PATH"
