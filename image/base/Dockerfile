# S2E container

FROM ubuntu:12.04
MAINTAINER Stefan Bucur <stefan.bucur@epfl.ch>

RUN apt-get update

# Tool dependencies
RUN apt-get install -y build-essential subversion git gettext python-docutils python-pygments nasm unzip

# Library dependencies
RUN apt-get install -y liblua5.1-dev libsdl1.2-dev libsigc++-2.0-dev binutils-dev

# LLVM dependencies
RUN apt-get build-dep -y llvm-3.1

# Qemu dependencies
RUN apt-get build-dep -y qemu

RUN mkdir -p /tmp/s2e
WORKDIR /tmp/s2e

# We build Z3 at this point because it needs root access to install
# and it doesn't need LLVM.
ADD http://download-codeplex.sec.s-msft.com/Download/SourceControlFileDownload.ashx?ProjectName=z3&changeSetId=dd62ca5eb36c2a62ee44fc5a79fc27c883de21ae /tmp/s2e/z3.zip

RUN unzip -d z3 z3.zip && rm z3.zip
RUN cd z3 && python scripts/mk_make.py && make -C build -j4 && make -C build install && cd .. && rm -rf z3/

ADD http://llvm.org/releases/3.2/llvm-3.2.src.tar.gz /tmp/s2e/
ADD http://llvm.org/releases/3.2/clang-3.2.src.tar.gz /tmp/s2e/
ADD http://llvm.org/releases/3.2/compiler-rt-3.2.src.tar.gz /tmp/s2e/

ADD llvm-3.2-memorytracer.patch /tmp/s2e/patches/
ADD clang-3.2-memorytracer.patch /tmp/s2e/patches/
ADD compiler-rt-3.2-asan4s2e.patch /tmp/s2e/patches/

RUN tar -xzvf llvm-3.2.src.tar.gz && \
  patch -d llvm-3.2.src -p0 -i ../patches/llvm-3.2-memorytracer.patch

RUN cp -r llvm-3.2.src llvm-3.2.src-native

RUN tar -xzvf clang-3.2.src.tar.gz && \
  patch -d clang-3.2.src -p0 -i ../patches/clang-3.2-memorytracer.patch && \
  mv clang-3.2.src llvm-3.2.src-native/tools/clang

RUN tar -xzvf compiler-rt-3.2.src.tar.gz && \
  patch -d compiler-rt-3.2.src -p0 -i ../patches/compiler-rt-3.2-asan4s2e.patch && \
  mv compiler-rt-3.2.src llvm-3.2.src-native/projects/compiler-rt
  
# LLVM native
RUN mkdir -p llvm-3.2-native
RUN cd llvm-3.2.src-native && ./configure --prefix=/opt/s2e/llvm-3.2-native \
  --enable-jit --enable-optimized --disable-assertions
RUN cd llvm-3.2.src-native &&\
  make ENABLE_OPTIMIZED=1 -j4 &&\
  make install &&\
  make clean

# LLVM
RUN mkdir -p llvm-3.2
RUN cd llvm-3.2.src && ./configure --prefix=/opt/s2e/llvm-3.2 \
  --enable-jit --enable-optimized \
  --target=x86_64 --enable-targets=x86 \
  CC=/opt/s2e/llvm-3.2-native/bin/clang \
  CXX=/opt/s2e/llvm-3.2-native/bin/clang++
RUN cd llvm-3.2.src &&\
  make ENABLE_OPTIMIZED=1 REQUIRES_RTTI=1 &&\
  make install &&\
  make clean

WORKDIR /
RUN rm -rf /tmp/s2e
