FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
RUN apt update --fix-missing
RUN apt install -y automake 
RUN apt install -y bison
RUN apt install -y    chrpath
RUN apt install -y g++
RUN apt install -y gcc
RUN apt install -y    git
RUN apt install -y    gperf
RUN apt install -y    gawk
RUN apt install -y    libexpat1-dev
RUN apt install -y    libncurses5-dev
RUN apt install -y    libsdl1.2-dev
RUN apt install -y    python2.7-dev
RUN apt install -y    texinfo
RUN apt install -y    unzip
RUN apt install -y    help2man
RUN apt install -y    make
RUN apt install -y    libtool-bin
RUN apt install -y    vim
RUN apt install -y    net-tools
RUN apt install -y iputils-ping
RUN apt install -y    flex

#add a user with bash as default terminal
RUN useradd -ms /bin/bash dummier
USER dummier
WORKDIR /home/dummier

CMD ["/bin/bash"]
