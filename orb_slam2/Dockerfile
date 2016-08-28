FROM ubuntu:14.04
MAINTAINER Eran Segal <eran@sphericalcow.io>

USER root
ARG NUM_CORES=2

## Install packages
RUN apt-get update && \
    apt-get install -y --force-yes --fix-missing --no-install-recommends \
    # Basics
    wget curl build-essential cmake g++ unzip git checkinstall yasm \
    # Required
    libeigen3-dev libglew-dev \
    # OpenCV Required
    libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev \
    # Optional
    libpython2.7-dev python-dev python-numpy libtbb2 libtbb-dev libv4l-dev v4l-utils \
    libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libavutil-dev libusb-1.0-0-dev
\
    # For reading still-image sequences 
    libjpeg-dev libpng12-dev libtiff5-dev libopenexr-dev \
    # Required by g2o
    libblas-dev liblapack-dev \
    # OpenGL
    freeglut3 freeglut3-dev

# Install FFMPEG (missing in Ubuntu 14.04)
WORKDIR /tmp
RUN git clone --depth 1 git://source.ffmpeg.org/ffmpeg && \
    cd ffmpeg && \
    ./configure --enable-shared --disable-static --enable-nonfree && \
    make -j ${NUM_CORES} && \
    make install && \
    make clean

# OpenCV
WORKDIR /tmp
RUN git clone --depth 1 git://github.com/opencv/opencv.git --branch 2.4.13 --single-branch && \
    cd opencv && \
    mkdir build && \
    cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
          -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D WITH_TBB=ON \
          -D BUILD_PYTHON_SUPPORT=ON \
          -D WITH_V4L=ON \
          -D WITH_QT=OFF \
          -D WITH_FFMPEG=YES \
          -D WITH_GSTREAMER=YES \
          -D WITH_TIFF=YES \
          -D WITH_OPENCL=YES \
          -D WITH_OPENGL=ON \
          ..

WORKDIR /tmp/opencv/build
RUN make -j $NUM_CORES && \
    make install && \
    echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf

# # libuvc: For cross-platform webcam video input via libusb
# WORKDIR /tmp
# RUN git clone --depth 1 git://github.com/ktossell/libuvc.git && \
#     cd libuvc && \
#     mkdir build && \
#     cd build && \
#     cmake .. && \
#     make -j $NUM_CORES && \
#     make install

# Pangolin
WORKDIR /tmp
RUN git clone --depth 1 git://github.com/stevenlovegrove/Pangolin.git && \
    cd Pangolin && \
    mkdir build && \
    cd build && \
    cmake -DCPP11_NO_BOOST=1 .. &&\
    make -j $NUM_CORES && \
    make install

# ORB_SLAM2
WORKDIR /home/user/
RUN git clone git://github.com/raulmur/ORB_SLAM2 && \
    cd ORB_SLAM2 && \
    #chmod +x build.sh && \
    #./build.sh
    # "Configuring and building Thirdparty/DBoW2 ..."
    cd Thirdparty/DBoW2 && \
    mkdir build && \
    cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make -j $NUM_CORES && \
    # "Configuring and building Thirdparty/g2o ..."
    cd ../../g2o && \
    mkdir build && \
    cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make -j $NUM_CORES && \
    cd ../../../ && \
    # "Uncompress vocabulary ..."
    cd Vocabulary && \
    tar -xzf ORBvoc.txt.tar.gz && \
    cd .. && \
    # "Configuring and building ORB_SLAM2 ..."
    mkdir build && \
    cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make -j $NUM_CORES

# cleanup
RUN ldconfig && \
    apt-get remove --purge -y curl build-essential checkinstall cmake && \
    apt-get autoclean && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

## Misc
ENV LC_ALL=C
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/user && \
    echo "user:x:${uid}:${gid}:Developer,,,:/home/user:/bin/bash" >> /etc/passwd && \
    echo "user:x:${uid}:" >> /etc/group && \
    echo "user ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/user && \
    chmod 0440 /etc/sudoers.d/user && \
    chown ${uid}:${gid} -R /home/user

## Use something like this to connect to your xserver
# -e DISPLAY -v $HOME/.Xauthority:/home/user/.Xauthority --net=host

USER user
ENV HOME /home/user
CMD /bin/bash