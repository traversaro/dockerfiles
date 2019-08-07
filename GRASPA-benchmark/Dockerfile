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
RUN curl https://packages.humanoids.kit.edu/h2t-key.pub | apt-key add -
RUN echo "deb http://packages.humanoids.kit.edu/bionic/main `lsb_release -cs` main" > /etc/apt/sources.list.d/armarx.list
RUN echo "deb http://packages.humanoids.kit.edu/bionic/testing `lsb_release -cs` testing" >> /etc/apt/sources.list.d/armarx.list
RUN apt update

# Install Qt5 and optional dependencies for qt5-powered
RUN apt install -y qt5-default libqt5designer5 libqt5designercomponents5

# Install Qt patch for Xfce
RUN apt install -y qml-module-qt-labs-folderlistmodel qml-module-qt-labs-settings

# Install simox dependencies
RUN apt install -y libboost-all-dev libeigen3-dev libnlopt-dev freeglut3-dev libqhull-dev liburdfdom-dev \
                   libassimp-dev h2t-libbullet h2t-libsoqt5-dev h2t-libsimage-dev h2t-libcoin80-dev

# Select global compilation options
ARG BUILD_TYPE=Release

# Build simox
RUN git clone https://gitlab.com/Simox/simox.git
# build directory is versioned
RUN cd simox/build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=$BUILD_TYPE -DSimDynamics_USE_BULLET=OFF \
    -DSimox_BUILD_SimDynamics=OFF && \
    make install
RUN rm -Rf simox

# Switch to gitpod user
RUN useradd -l -u 33333 -G sudo -md /home/gitpod -s /bin/bash -p gitpod gitpod && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers
USER gitpod

# Install python packages
RUN pip3 install numpy numpy-stl opencv-python opencv-contrib-python scipy

# Set up .bashrc
RUN echo "PS1='\[\e]0;\u \w\a\]\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\] \$ '" >> ~/.bashrc

# Switch back to root
USER root

# Set environmental variables
ENV DISPLAY=:1

# Set up script to launch graphics and vnc
COPY start-vnc-session.sh /usr/bin/start-vnc-session.sh
RUN chmod +x /usr/bin/start-vnc-session.sh

# Manage x11vnc port
EXPOSE 5901

# Manage noVNC port
EXPOSE 6080

# Launch bash from /workspace
WORKDIR /workspace
CMD ["bash"]
