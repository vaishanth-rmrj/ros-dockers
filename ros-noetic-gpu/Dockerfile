FROM nvidia/cuda:11.0.3-base-ubuntu20.04

LABEL Description="ROS: Noetic-Desktop (Ubuntu 20.04)"
# LABEL com.turlucode.ros.version="noetic"

# Install packages without prompting the user to answer any questions
ENV DEBIAN_FRONTEND noninteractive 

# Install packages
RUN apt-get update && apt-get install -y \
locales \
lsb-release \
mesa-utils \
git \
subversion \
nano \
terminator \
xterm \
wget \
curl \
htop \
libssl-dev \
build-essential \
dbus-x11 \
software-properties-common \
nvidia-cuda-toolkit \
gdb valgrind && \
apt-get clean && rm -rf /var/lib/apt/lists/*

# Install cmake 3.22.1
RUN git clone https://gitlab.kitware.com/cmake/cmake.git && \
cd cmake && git checkout tags/v3.22.1 && ./bootstrap --parallel=8 && make -j8 && make install && \
cd .. && rm -rf cmake

# Install tmux 3.2
RUN apt-get update && apt-get install -y automake autoconf pkg-config libevent-dev libncurses5-dev bison && \
apt-get clean && rm -rf /var/lib/apt/lists/*
RUN git clone https://github.com/tmux/tmux.git && \
cd tmux && git checkout tags/3.2 && ls -la && sh autogen.sh && ./configure && make -j8 && make install

# Install new paramiko (solves ssh issues)
RUN apt-add-repository universe
RUN apt-get update && apt-get install -y python3-pip python3 build-essential && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN /usr/bin/yes | pip3 install --upgrade pip
RUN /usr/bin/yes | pip3 install --upgrade virtualenv
RUN /usr/bin/yes | pip3 install --upgrade paramiko
RUN /usr/bin/yes | pip3 install --ignore-installed --upgrade numpy protobuf

# Locale
RUN locale-gen en_US.UTF-8  
ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en  
ENV LC_ALL en_US.UTF-8

# Install ROS
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN apt-get update && apt-get install -y --allow-downgrades --allow-remove-essential --allow-change-held-packages \
libpcap-dev \
libopenblas-dev \
gstreamer1.0-tools libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev \
ros-noetic-desktop-full python3-rosdep python3-rosinstall-generator python3-vcstool build-essential \
ros-noetic-socketcan-bridge \
ros-noetic-geodesy && \
apt-get clean && rm -rf /var/lib/apt/lists/*

# Configure ROS
RUN rosdep init && rosdep update 
RUN echo "source /opt/ros/noetic/setup.bash" >> /root/.bashrc
RUN echo "export ROSLAUNCH_SSH_UNKNOWN=1" >> /root/.bashrc

WORKDIR /

# installing pytorch (optional)
RUN pip3 install -f torch torchvision

# installing required packages (optional)
RUN pip3 install pandas
RUN pip3 install matplotlib
RUN pip3 install scikit-learn
RUN pip3 install seaborn
RUN pip3 install opencv-python

# Terminator Config (optional)
RUN mkdir -p /root/.config/terminator/
COPY terminator_config /root/.config/terminator/config 

# Entry script
COPY entrypoint_setup.sh / 
RUN chmod +x /entrypoint_setup.sh
ENTRYPOINT ["/entrypoint_setup.sh"]

# Launch terminator
CMD ["terminator"]