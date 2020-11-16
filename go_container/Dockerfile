FROM golang:latest

RUN uname -a

LABEL maintainer="Carlos Sanchez <sanchezcl@gmail.com>"

RUN export PATH="$HOME/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/bin:/bin:/golib"

RUN apt-get update -yqq \
    && apt-get install -yqq git unzip

WORKDIR /app

# Start as root
USER root

###########################################################################
# Golang non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupadd -g ${PGID} gonlang && \
    useradd -u ${PUID} -g gonlang -m gonlang && \
    usermod -p "*" gonlang -s /bin/bash

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# PHP OCI8:
###########################################################################

USER root

ENV LD_LIBRARY_PATH="/opt/oracle/instantclient_12_1"
ENV OCI_HOME="/opt/oracle/instantclient_12_1"
ENV OCI_LIB_DIR="/opt/oracle/instantclient_12_1"
ENV OCI_INCLUDE_DIR="/opt/oracle/instantclient_12_1/sdk/include"
ENV OCI_VERSION=12

  # Install wget
RUN  apt-get update && apt-get install --no-install-recommends -y wget \
  # Install Oracle Instantclient
  && mkdir /opt/oracle \
      && cd /opt/oracle \
      && wget https://github.com/diogomascarenha/oracle-instantclient/raw/master/instantclient-basic-linux.x64-12.1.0.2.0.zip \
      && wget https://github.com/diogomascarenha/oracle-instantclient/raw/master/instantclient-sdk-linux.x64-12.1.0.2.0.zip \
      && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
      && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
      && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
      && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
      && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
      && rm -rf /opt/oracle/*.zip \
  # Install PHP extensions deps
  && apt-get update \
      && apt-get install --no-install-recommends -y \
          libaio-dev

COPY ./oci8.pc /usr/lib/pkgconfig/oci8.pc

ENV LD_LIBRARY_PATH /usr/lib:/usr/local/lib:/opt/oracle/instantclient_12_1
RUN go get -u github.com/mattn/go-oci8

###########################################################################
# ping:
###########################################################################

USER root

RUN apt-get update -yqq && \
    apt-get -y install inetutils-ping

###########################################################################
# Bash Git Prompt
###########################################################################

ARG INSTALL_GIT_PROMPT=false

COPY git-prompt.sh /tmp/git-prompt

RUN git clone https://github.com/magicmonty/bash-git-prompt.git /root/.bash-git-prompt --depth=1 && \
    cat /tmp/git-prompt >> /root/.bashrc && \
    rm /tmp/git-prompt

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/gonlang/aliases.sh
COPY ./aliases.sh /root/.bash_aliases
COPY ./aliases.sh /home/laradock/.bash_aliases

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/gonlang/aliases.sh && \
    chown gonlang:gonlang /home/gonlang/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

USER gonlang

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
    echo "# Load GOPATH" >> ~/.bashrc && \
	echo "export GOPATH=/golib" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

###########################################################################
# Final touch
###########################################################################

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

CMD tail -f /dev/stdout