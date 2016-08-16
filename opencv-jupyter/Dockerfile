FROM jupyter/scipy-notebook

MAINTAINER Eran Segal <eran@sphericalcow.io>
# Sources:
# - http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html
# - https://hub.docker.com/r/hudl/opencv/~/dockerfile/
# - https://github.com/trafferty/docker-ipython-opencv/blob/master/Dockerfile
# - https://github.com/srzyhead/docker-ipython-opencv/commit/d5e99f7c8986723ed85d90d9cf95267b866e1017

ENV NUM_CORES 2

USER root

# Install opencv prerequisites...
RUN apt-get update -qq && \
    apt-get install -y --force-yes \
    build-essential \
    cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev \
    python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev

# Install more stuff...
RUN apt-get install -y --force-yes --fix-missing \
    curl \
    g++ \
    autoconf \
    automake \
    checkinstall \
    yasm \
    libtiff5-dev \
    libxine2-dev \
    libgstreamer0.10-dev \
    libgstreamer-plugins-base0.10-dev \
    libv4l-dev \
    libmp3lame-dev \
    libopencore-amrnb-dev \
    libopencore-amrwb-dev \
    libtheora-dev \
    libvorbis-dev \
    libxvidcore-dev \
    libtool \
    v4l-utils \
    default-jdk \
    wget \
    tmux \
    libqt4-dev \
    libphonon-dev \
    libxml2-dev \
    libxslt1-dev \
    qtmobility-dev \
    libqtwebkit-dev \
    unzip \
    libboost-all-dev \
    libeigen3-dev \
    libhdf5-serial-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


WORKDIR /usr/local/src
RUN git clone --depth 1 git://source.ffmpeg.org/ffmpeg
RUN git clone --depth 1 https://github.com/opencv/opencv.git
RUN git clone --depth 1 https://github.com/opencv/opencv_contrib.git
RUN mkdir -p opencv/release

# Install FFMPEG (due to compilation errors of OpenCV)
# see: http://stackoverflow.com/questions/26592577/installing-opencv-in-ubuntu-14-10/27020828#27020828
WORKDIR /usr/local/src/ffmpeg
RUN ./configure --enable-shared --disable-static --enable-nonfree \
    && make -j ${NUM_CORES} \
    && make install    

# Build OpenCV 3.x
WORKDIR /usr/local/src/opencv/release
RUN cmake -D CMAKE_BUILD_TYPE=RELEASE \
          -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D WITH_TBB=ON \
          -D BUILD_PYTHON_SUPPORT=ON \
          -D WITH_V4L=ON \
          # bug w/ tag=3.1.0: cmake has error
          -D INSTALL_C_EXAMPLES=OFF \
          -D INSTALL_PYTHON_EXAMPLES=ON \
          -D BUILD_EXAMPLES=ON \
          -D BUILD_DOCS=ON \
          -D OPENCV_EXTRA_MODULES_PATH=/usr/local/src/opencv_contrib/modules \
          -D WITH_XIMEA=YES \
#          -D WITH_QT=YES \
          -D WITH_FFMPEG=YES \
          -D WITH_PVAPI=YES \
          -D WITH_GSTREAMER=YES \
          -D WITH_TIFF=YES \
          -D WITH_OPENCL=YES \
          -D PYTHON2_EXECUTABLE=/opt/conda/envs/python2/bin/python \
          -D PYTHON2_INCLUDE_DIR=/opt/conda/envs/python2/include/python2.7 \
          -D PYTHON2_LIBRARIES=/opt/conda/envs/python2/lib/libpython2.7.so \
          -D PYTHON2_PACKAGES_PATH=/opt/conda/envs/python2/lib/python2.7/site-packages \
          -D PYTHON2_NUMPY_INCLUDE_DIRS=/opt/conda/envs/python2/lib/python2.7/site-packages/numpy/core/include/ \
          -D BUILD_opencv_python3=ON \
          -D PYTHON3_EXECUTABLE=/opt/conda/bin/python \
          -D PYTHON3_INCLUDE_DIR=/opt/conda/include/python3.5m/ \
          -D PYTHON3_LIBRARY=/opt/conda/lib/libpython3.so \
          -D PYTHON_LIBRARY=/opt/conda/lib/libpython3.so \
          -D PYTHON3_PACKAGES_PATH=/opt/conda/lib/python3.5/site-packages \
          -D PYTHON3_NUMPY_INCLUDE_DIRS=/opt/conda/lib/python3.5/site-packages/numpy/core/include/ \
          ..

RUN make -j$NUM_CORES \
    && make install

RUN sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf' && ldconfig

## Additional modules
WORKDIR /usr/local/src

# GSL 2.1
RUN curl -L ftp://ftp.gnu.org/gnu/gsl/gsl-2.1.tar.gz | tar zxf - && \
    cd gsl-2.1 && \
    ./configure && \
    make && make install && \
    ldconfig && \
    cd .. && \
    rm -rf gsl-2.1

# VLFeat
RUN wget http://www.vlfeat.org/download/vlfeat-0.9.20-bin.tar.gz && \
    tar zxvf vlfeat-0.9.20-bin.tar.gz && \
    rm vlfeat-0.9.20-bin.tar.gz
ENV VLFEAT_DIR=/usr/local/src/vlfeat-0.9.20

# fix libdc1394 error: Failed to initialize libdc1394
RUN ln /dev/null /dev/raw1394
RUN echo 'KERNEL=="raw1394", GROUP="video"' > /etc/udev/rules.d/raw1394.rules

## prepare the notebooks folder
RUN mkdir /notebooks
RUN chown -R jovyan:users /notebooks
VOLUME /notebooks
WORKDIR /notebooks

## Misc
ENV LC_ALL=C

## Switch back to jupyter user
USER jovyan

## Additional python modules
#RUN /opt/conda/envs/python2/bin/pip install imutils minieigen
#RUN /opt/conda/bin/pip install imutils minieigen
