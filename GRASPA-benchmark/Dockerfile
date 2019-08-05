FROM ubuntu:latest
LABEL maintainer="ugo.pattacini@iit.it"

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Update apt database
RUN apt update

# Install essentials
RUN apt install -y apt-utils software-properties-common sudo psmisc lsb-release \
                   tmux nano wget curl build-essential git gitk cmake cmake-curses-gui \
                   libedit-dev libxml2-dev autoconf                   

# Install graphics
RUN apt install -y xfce4 xfce4-goodies xserver-xorg-video-dummy xserver-xorg-legacy firefox
RUN sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY xorg.conf /etc/X11/xorg.conf

# We need the most recent x11vnc
RUN sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list && \
    apt update
RUN git clone https://github.com/LibVNC/x11vnc.git /opt/x11vnc && \
    cd /opt/x11vnc && \
    apt build-dep -y x11vnc && \
    autoreconf -fiv && \
    ./configure && \
    make && make install && \
    rm -Rf /opt/x11vnc

# Install Mesa for GPU emulation
RUN add-apt-repository ppa:ubuntu-x-swat/updates -y && \
    apt dist-upgrade -y && \
    apt install -y mesa-utils

# Install terminator
RUN apt install -y terminator

# Install python3
RUN add-apt-repository ppa:deadsnakes/ppa -y && \
    apt dist-upgrade -y && \
    apt install -y python3 python3-dev python3-pip python3-setuptools

# Install websockify
RUN git clone https://github.com/novnc/websockify.git /opt/websockify && \
    cd /opt/websockify && \
    python3 setup.py install

# Clone noVNC
RUN git clone https://github.com/novnc/noVNC.git /opt/novnc
RUN echo "<html><head><meta http-equiv=\"Refresh\" content=\"0; url=vnc.html\"></head></html>" > /opt/novnc/index.html

# Acquire custom packages
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 57A5ACB6110576A6
RUN curl https://packages.humanoids.kit.edu/h2t-key.pub | apt-key add -
RUN echo "deb http://www.icub.org/ubuntu `lsb_release -cs` contrib/science" > /etc/apt/sources.list.d/icub.list
RUN echo "deb http://packages.humanoids.kit.edu/bionic/main `lsb_release -cs` main" > /etc/apt/sources.list.d/armarx.list
RUN echo "deb http://packages.humanoids.kit.edu/bionic/testing `lsb_release -cs` testing" >> /etc/apt/sources.list.d/armarx.list
RUN apt update

# Install yarp dependencies
RUN apt install -y icub-common

# Install Qt patch for Xfce and optional dependencies for qt5-powered
RUN apt install -y qml-module-qt-labs-folderlistmodel qml-module-qt-labs-settings \
                   libqt5designer5 libqt5designercomponents5

# Install simox dependencies
RUN apt install -y libboost-all-dev libeigen3-dev libnlopt-dev freeglut3-dev libqhull-dev liburdfdom-dev \
                   h2t-libbullet h2t-libsoqt5-dev h2t-libsimage-dev h2t-libcoin80-dev
RUN pip3 install numpy numpy-stl opencv-python opencv-contrib-python scipy 

# Select global compilation options
ARG BRANCH=devel
ARG BUILD_TYPE=Release
ARG OpenCV_TAG=3.4.0

# Build OpenCV with aruco module
RUN git clone https://github.com/opencv/opencv.git -b $OpenCV_TAG /opt/opencv && \
    git clone https://github.com/opencv/opencv_contrib.git -b $OpenCV_TAG /opt/opencv_contrib
RUN mkdir /opt/opencv_install && mkdir /opt/opencv/build && cd /opt/opencv/build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE \
    -DCMAKE_INSTALL_PREFIX=/opt/opencv_install \
    -DOPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
    -DBUILD_opencv_aruco=ON && \
    make install
RUN rm -Rf /opt/opencv /opt/opencv_contrib
ENV OpenCV_DIR=/opt/opencv_install

# Build ycm
RUN git clone https://github.com/robotology/ycm.git -b $BRANCH
RUN cd ycm && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE && \
    make install
RUN rm -Rf ycm

# Build yarp
RUN git clone https://github.com/robotology/yarp.git -b $BRANCH
RUN cd yarp && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE \
    -DYARP_COMPILE_libYARP_math=ON \
    -DYARP_COMPILE_GUIS=ON \
    -DYARP_COMPILE_DEVICE_PLUGINS=OFF && \
    make install
RUN cp yarp/scripts/yarp_completion /etc/bash_completion.d/.
RUN rm -Rf yarp

# Build icub-main
RUN git clone https://github.com/robotology/icub-main.git -b $BRANCH
RUN cd icub-main && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE \
    -DENABLE_icubmod_cartesiancontrollerserver=ON \
    -DENABLE_icubmod_cartesiancontrollerclient=ON \
    -DENABLE_icubmod_gazecontrollerclient=ON && \
    make install
RUN rm -Rf icub-main

# Build simox
RUN git clone https://gitlab.com/Simox/simox.git
RUN cd simox && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE -DSimDynamics_USE_BULLET=off \
    -DSimDynamics_USE_BULLET_USE_GLU=off -DSimox_BUILD_Simdynamics=off \
    -DSimox_USE_QT5=ON && \
    make install
RUN rm -Rf simox

# Switch to gitpod user
RUN useradd -l -u 33333 -G sudo -md /home/gitpod -s /bin/bash -p gitpod gitpod && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers
USER gitpod

# Build icub-contrib-common
WORKDIR /home/gitpod
RUN mkdir iCubContrib
RUN git clone https://github.com/robotology/icub-contrib-common.git
RUN cd icub-contrib-common && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_INSTALL_PREFIX=/home/gitpod/iCubContrib && \
    make install
RUN rm -Rf icub-contrib-common
ENV ICUBcontrib_DIR=/home/gitpod/iCubContrib

# Set up .bashrc
RUN echo "PS1='\[\e]0;\u \w\a\]\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\] \$ '" >> ~/.bashrc
RUN echo "export PATH=$PATH:/home/gitpod/iCubContrib/bin" >> ~/.bashrc

# Switch back to root
USER root

# Set environmental variables
ENV DISPLAY=:1
ENV YARP_DATA_DIRS=/usr/local/share/yarp:/usr/local/share/iCub:/home/gitpod/iCubContrib/share/ICUBcontrib
ENV LD_LIBRARY_PATH=/usr/local/lib/yarp:/usr/local/lib/robottestingframework

# Set up script to launch graphics and vnc
COPY start-vnc-session.sh /usr/bin/start-vnc-session.sh
RUN chmod +x /usr/bin/start-vnc-session.sh

# Manage x11vnc port
EXPOSE 5901

# Manage noVNC port
EXPOSE 6080

# Manage yarp port
EXPOSE 10000/tcp 10000/udp

# Launch bash from /workspace
WORKDIR /workspace
CMD ["bash"]
