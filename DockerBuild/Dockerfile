FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
RUN apt update
RUN apt install -y automake \
    bison \
    chrpath \
    flex \
    g++ \
    git \
    gperf \
    gawk \
    libexpat1-dev \
    libncurses5-dev \
    libsdl1.2-dev \
    python2.7-dev \
    texinfo \
    unzip   \
    help2man    \
    make    \
    libtool* \
    vim \
    net-tools

#add a user with bash as default terminal
RUN useradd -ms /bin/bash dummier
USER dummier
WORKDIR /home/dummier

CMD ["/bin/bash"]
