FROM stefanbucur/s2e-base:latest
MAINTAINER Stefan Bucur <stefan.bucur@epfl.ch>

RUN apt-get update &&\
  apt-get install -y emacs23-nox sudo

# Additional S2E dependencies
RUN apt-get install -y libdwarf-dev libelf-dev libboost-dev \
  libsqlite3-dev libmemcached-dev libboost-serialization-dev \
  libboost-system-dev libc6-dev-i386

# Install protobuf
RUN mkdir -p /build &&\
  wget 'https://protobuf.googlecode.com/svn/rc/protobuf-2.6.0.tar.gz' &&\
  tar xzvf protobuf-2.6.0.tar.gz &&\
  cd protobuf-2.6.0 && ./configure && make -j8 && make install && ldconfig &&\
  rm -rf /build

RUN mkdir -p /home/s2e
WORKDIR /home/s2e

# Create an image user
# Tip from http://www.projectatomic.io/docs/docker-image-author-guidance/ 
RUN groupadd -r s2e -g 433 && \
  useradd -u 431 -r -g s2e -d /home/s2e -s /sbin/nologin -c "S2E image user" s2e
  
RUN chown -R s2e:s2e /home/s2e

USER s2e
