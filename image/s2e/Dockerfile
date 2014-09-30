FROM stefanbucur/s2e-base
MAINTAINER Stefan Bucur <stefan.bucur@epfl.ch>

RUN apt-get update &&\
  apt-get install -y emacs23-nox

RUN mkdir -p /home/s2e
WORKDIR /home/s2e

# Create an image user
# Tip from http://www.projectatomic.io/docs/docker-image-author-guidance/ 
RUN groupadd -r s2e -g 433 && \
  useradd -u 431 -r -g s2e -d /home/s2e -s /sbin/nologin -c "S2E image user" s2e
  
RUN chown -R s2e:s2e /home/s2e

USER s2e
