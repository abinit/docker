FROM ubuntu:focal

ARG LIBXC=4.3.4
ARG ABINIT=9.6.2

USER root

SHELL ["/bin/bash","-c"]

ENV DEBIAN_FRONTEND="noninteractive" 

RUN useradd abinit --create-home --home /home/abinit --shell /bin/bash
RUN chown abinit:abinit /home/abinit

RUN apt update
RUN apt upgrade -y 

RUN apt install -y wget software-properties-common

RUN add-apt-repository ppa:piti-diablotin/abiout -y

RUN apt -y install \
  agate \
  qagate \
  tar \
  make \
  gfortran \
  gcc \
  g++ \
  patch \
  libopenmpi-dev \
  openmpi-bin \
  libfftw3-3 \
  libfftw3-dev \
  libatlas3-base \
  libatlas-base-dev \
  liblapack-dev \
  libscalapack-openmpi-dev \
  libgsl-dev \
  netcdf-bin \
  libnetcdf-dev \
  libnetcdf-mpi-dev \
  libnetcdff-dev \
  libxml2-dev \
  gnuplot \
  ffmpeg \
  python-is-python3 \
  python3-pip \
  cmake 

WORKDIR /tmp/

RUN wget -O libxc-$LIBXC.tar.gz "http://www.tddft.org/programs/libxc/down.php?file=$LIBXC/libxc-$LIBXC.tar.gz"
RUN tar xfz libxc-$LIBXC.tar.gz 
RUN cd libxc-$LIBXC/ && \
      ./configure --prefix=/opt/libxc-$LIBXC --enable-static --disable-shared &&\
      make -j $(nproc --all) && \
      make install 

RUN wget --no-check-certificate -O abinit-$ABINIT.tar.gz "https://www.abinit.org/sites/default/files/packages/abinit-$ABINIT.tar.gz" 
RUN tar xfz abinit-$ABINIT.tar.gz 
RUN cd abinit-$ABINIT && \
      mkdir build && \
      cd build/ && \
      ../configure \
      FC=mpif90 CC=mpicc CPP=cpp \
      --enable-gw-dpc=yes \
      --with-optim-flavor="standard" \
      --with-debug-flavor="none" \
      --enable-openmp=no \
      --with-mpi=yes \
      --with-mpi-flavor=native \
      --enable-mpi-inplace=yes \
      --enable-mpi-io=yes \
      --with-netcdf=yes \
      --with-hdf5=yes \
      --with-netcdf-fortran=yes \
      --with-libxc="/opt/libxc-$LIBXC" \
      --with-fft-flavor="fftw3" \
      FFTW3_CPPFLAGS="-I/usr/include" \
      FFTW3_FCFLAGS="-I/usr/include" \
      FFTW3_LIBS="-L/usr/lib -lfftw3 -lfftw3f" \
      --with-libxml2=yes \
      --prefix=/opt/abinit-$ABINIT && \
      make -j $(nproc --all) && \
      make install 

RUN rm -rf libxc* abinit*

COPY requirements_abipy.txt .

RUN pip install -r requirements_abipy.txt

RUN rm -rf requirements_abipy.txt

WORKDIR /home/abinit

USER abinit

RUN echo "export PATH=$PATH:/opt/abinit-$ABINIT/bin:/opt/libxc-$LIBXC/bin" >> ~abinit/.bashrc
