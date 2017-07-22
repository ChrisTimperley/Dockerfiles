FROM ubuntu:16.04
MAINTAINER Chris Timperley "christimperley@gmail.com"

# Install basic dependencies
# - we install binutils-gold to speed up compilation
# TODO: install Ninja
RUN apt-get update && \
    apt-get install -y  subversion \
                        vim \
                        git \
                        g++ \
                        python \
                        perl \
                        cmake \
                        unzip \
                        wget \
                        curl \
                        build-essential \
                        bison \
                        flex \
                        binutils-gold \
                        libcap-dev \
	                      libncurses5-dev \
                        python-minimal \
                        python-pip \
                        groff \
                        libboost-all-dev \
                        zlib1g-dev

ENV CXX=g++

# Download a different version of LLVM/Clang for use with KLEE
ENV KLEE_LOCATION /opt/klee
RUN mkdir -p "${KLEE_LOCATION}" && \
    git clone https://github.com/llvm-mirror/llvm.git "${KLEE_LOCATION}/llvm" && \
	  cd "${KLEE_LOCATION}/llvm" && \
    git checkout release_34 && \
    mkdir build && \
    cd tools && \
	  git clone https://github.com/llvm-mirror/clang.git && \
    cd clang && \
    git checkout release_34 && \
	  cd ../../build && \
    ../configure --enable-optimized --prefix="${KLEE_LOCATION}" && \
    make -j4 && \
    make install && \
    cd / && \
    rm -rf /tmp/*

# Install minisat
# WARNING: this isn't fixed to a particular version!
RUN git clone https://github.com/stp/minisat.git /tmp/minisat && \
    cd /tmp/minisat && \
	  mkdir build && \
    cd build && \
    cmake -DSTATIC_BINARIES=ON -DCMAKE_INSTALL_PREFIX="${KLEE_LOCATION}" ../ && \ 
	  make -j install && \
    cd / && \
    rm -rf /tmp/*

# Install stp
RUN cd "${KLEE_LOCATION}" && \
    wget --quiet https://github.com/stp/stp/archive/2.1.2.tar.gz && \
    tar -xf 2.1.2.tar.gz && \
    rm 2.1.2.tar.gz && \
    mv stp-2.1.2 stp && \
    cd stp && \ 
    mkdir build && \
    cd build && \
	  cmake -DSTATIC_BINARIES=ON \
          -DBUILD_SHARED_LIBS:BOOL=OFF \
          -DENABLE_PYTHON_INTERFACE:BOOL=OFF \
          -DCMAKE_INSTALL_PREFIX="${KLEE_LOCATION}" .. && \
    make -j && \
    make install && \
    ulimit -s unlimited && \
    rm -rf /tmp/*

# Install uclibc
# WARNING: this isn't fixed to a particular version!
# TODO: this is hanging on to more than it ought to
RUN git clone https://github.com/klee/klee-uclibc.git "${KLEE_LOCATION}/uclibc" && \
	  cd "${KLEE_LOCATION}/uclibc" && \
    ./configure --with-llvm-config "${KLEE_LOCATION}/bin/llvm-config" \
                --make-llvm-lib && \
    make -j && \
    rm -rf .git

# Install KLEE
#
# TODO: this is a problem! We're going to end up adding two versions of LLVM to
#       the PATH
ENV PATH="${KLEE_LOCATION}/bin:${PATH}"
ENV LLVM_COMPILER clang
RUN git clone https://github.com/klee/klee.git /tmp/klee && \
    cd /tmp/klee && \
	  ./configure prefix="${KLEE_LOCATION}" \
                --with-stp="${KLEE_LOCATION}/stp" \
                --with-uclibc="${KLEE_LOCATION}/uclibc" \
                --enable-posix-runtime \
                LDFLAGS="-L${KLEE_LOCATION}/lib" \
                CPPFLAGS="-I${KLEE_LOCATION}/include" && \
	  make install -j && \
    cd / && \
    rm -rf /tmp/*
#ENV PATH="/opt/KLEE/KLEE/Release+Asserts/bin:${PATH}"

# Install Z3 from source
#
# WARNING: this isn't tied to a particular version!
ENV Z3_LOCATION /opt/z3
RUN git clone https://github.com/Z3Prover/z3.git /tmp/z3 && \
    cd /tmp/z3 && \
	  python scripts/mk_make.py && \
    cd build && \
	  make -j4 && \
    make install PREFIX="${Z3_LOCATION}" && \
    cd / && \
    rm -rf /tmp/*

# install wllvm
RUN pip install --upgrade pip && \
    pip install --upgrade wllvm

# coreutils
RUN wget -nv https://ftp.gnu.org/gnu/coreutils/coreutils-6.11.tar.gz && \
    tar -xf coreutils-6.11.tar.gz && \
    rm coreutils-6.11.tar.gz && \
    mv coreutils-6.11 coreutils
ENV CC wllvm
RUN cd coreutils && \
    mkdir obj-llvm && \
    cd obj-llvm

#CFLAGS="-g -fprofile-arcs -ftest-coverage"
RUN cd coreutils/obj-llvm && \
    ../configure --disable-nls CFLAGS="-g" && \
    make -j$(nproc)
RUN cd coreutils/obj-llvm/src && \
    find . -executable -type f | xargs -I '{}' extract-bc '{}'

# export Z3 and KLEE
VOLUME /opt/z3
VOLUME /opt/klee
