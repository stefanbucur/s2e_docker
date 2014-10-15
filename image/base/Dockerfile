# S2E base container

FROM ubuntu:12.04
MAINTAINER Stefan Bucur <stefan.bucur@epfl.ch>

# Tool dependencies
RUN apt-get update && apt-get install -y build-essential subversion git gettext python-docutils python-pygments nasm unzip wget

# Library dependencies
RUN apt-get install -y liblua5.1-dev libsdl1.2-dev libsigc++-2.0-dev binutils-dev

# LLVM dependencies
RUN apt-get build-dep -y llvm-3.1

# Qemu dependencies
RUN apt-get build-dep -y qemu

# Build & install Z3 (doesn't need LLVM)
RUN mkdir -p /build && cd /build &&\
  wget -O z3.zip 'http://download-codeplex.sec.s-msft.com/Download/SourceControlFileDownload.ashx?ProjectName=z3&changeSetId=dd62ca5eb36c2a62ee44fc5a79fc27c883de21ae' &&\
  unzip -d z3 z3.zip && rm z3.zip &&\
  cd z3 &&\
  python scripts/mk_make.py && make -C build -j4 && make -C build install &&\
  rm -rf /build

ADD llvm-3.2-memorytracer.patch /opt/s2e/llvm/patches
ADD clang-3.2-memorytracer.patch /opt/s2e/llvm/patches
ADD compiler-rt-3.2-asan4s2e.patch /opt/s2e/llvm/patches

# Download & patch the LLVM source code
RUN cd /opt/s2e/llvm &&\
  wget 'http://llvm.org/releases/3.2/llvm-3.2.src.tar.gz' &&\
  wget 'http://llvm.org/releases/3.2/clang-3.2.src.tar.gz' &&\
  wget 'http://llvm.org/releases/3.2/compiler-rt-3.2.src.tar.gz' &&\
  tar -xzvf llvm-3.2.src.tar.gz &&\
  patch -d llvm-3.2.src -p0 -i ../patches/llvm-3.2-memorytracer.patch &&\
  cp -r llvm-3.2.src llvm-3.2.src-native &&\
  tar -xzvf clang-3.2.src.tar.gz &&\
  patch -d clang-3.2.src -p0 -i ../patches/clang-3.2-memorytracer.patch && \
  mv clang-3.2.src llvm-3.2.src-native/tools/clang &&\
  tar -xzvf compiler-rt-3.2.src.tar.gz &&\
  patch -d compiler-rt-3.2.src -p0 -i ../patches/compiler-rt-3.2-asan4s2e.patch && \
  mv compiler-rt-3.2.src llvm-3.2.src-native/projects/compiler-rt &&\
  rm -f *.src.tar.gz && rm -rf patches/

# Build & install the native LLVM
RUN cd /opt/s2e/llvm/llvm-3.2.src-native &&\
  ./configure \
    --prefix=/opt/s2e/llvm/llvm-3.2-native \
    --enable-jit --enable-optimized --disable-assertions &&\
  make ENABLE_OPTIMIZED=1 -j4 && make install && cd .. && rm -rf llvm-3.2.src-native

# Build the target LLVM
RUN cd /opt/s2e/llvm && mkdir -p llvm-3.2.build && cd llvm-3.2.build &&\
  ../llvm-3.2.src/configure \
    --enable-jit --enable-optimized --target=x86_64 --enable-targets=x86 \
    CC=/opt/s2e/llvm/llvm-3.2-native/bin/clang \
    CXX=/opt/s2e/llvm/llvm-3.2-native/bin/clang++ &&\
  make ENABLE_OPTIMIZED=1 REQUIRES_RTTI=1

# At this point we're left with three directories in /opt/s2e/llvm:
# - llvm-3.2-native
# - llvm-3.2.src
# - llvm-3.2.build
