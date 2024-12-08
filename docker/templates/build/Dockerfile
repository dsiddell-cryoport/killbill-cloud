FROM ubuntu:20.04
LABEL maintainer="killbilling-users@googlegroups.com"

USER root

ENV LC_CTYPE=en_US.UTF-8
ENV LC_ALL=C
ENV LANG=en_US.UTF-8
ENV PYTHONIOENCODING=utf8

# Install build dependencies and convenient utilities
# https://github.com/moby/moby/issues/4032
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
      # https://github.com/tianon/docker-brew-ubuntu-core/issues/59
      #adduser \
      apt-utils \
      build-essential \
      curl \
      git \
      less \
      libaio1 \
      libapr1 \
      libcurl4 \
      libcurl4-openssl-dev \
      libmysqlclient-dev \
      libnuma1 \
      libpq-dev \
      maven \
      mysql-client \
      net-tools \
      openjdk-11-jdk-headless \
      postgresql-client \
      python3-lxml \
      software-properties-common \
      sudo \
      telnet \
      unzip \
      zip \
      vim && \
    apt-add-repository -y ppa:rael-gc/rvm && \
    rm -rf /var/lib/apt/lists/*

# Configure default JAVA_HOME path
RUN update-java-alternatives -s java-1.11.0-openjdk-$(dpkg --print-architecture)
RUN rm -f /usr/lib/jvm/default-java && ln -s java-11-openjdk-$(dpkg --print-architecture) /usr/lib/jvm/default-java
ENV JAVA_HOME=/usr/lib/jvm/default-java
ENV JSSE_HOME=$JAVA_HOME/jre/

# Create killbill user into sudo group and reinitialize the password
ENV KILLBILL_HOME=/var/lib/killbill
RUN adduser killbill \
            --home $KILLBILL_HOME \
            --disabled-password \
            --gecos '' && \
    usermod -aG sudo killbill && \
    echo "killbill:killbill" | chpasswd && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

#RUN adduser 10001 \
#            --home $KILLBILL_HOME \
#            --disabled-password \
#            --gecos '' && \
#            --force-badname #\
#    usermod -aG sudo 10001 && \
#    echo "10001:10001" | chpasswd && \
#    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

RUN useradd -u 10001 -m -d ${KILLBILL_HOME} -s /bin/bash --badname 10001 || true && \
    #chown -R 10001:10001 ${KILLBILL_HOME} \
    echo "10001 ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
#RUN apt-get update && apt-get install -y sudo && \
#    useradd -u 10001 -m -d ${KILLBILL_HOME} -s /bin/bash --badname 10001 || true && \
#    usermod -aG sudo 10001
#RUN adduser 10001 --uid 10001 --home $KILLBILL_HOME --disabled-password --gecos '' --force-badname

# Add a non-root user with UID and GID between 10000 and 20000
#RUN groupadd -g 10001 killbill && useradd -u 10001 -g 10001 -d ${KILLBILL_HOME} killbill \
#    && chown -R killbill:killbill ${KILLBILL_HOME}

#RUN chown -R 10001:10001 ${KILLBILL_HOME}

# Install RVM
RUN apt-get update && \
    apt-get install --no-install-recommends -y rvm && \
    rm -rf /var/lib/apt/lists/*

# Add extra rubies
RUN /bin/bash -l -c "rvm install 2.2.2 && rvm install 2.4.2 && rvm install jruby-9.1.14.0"

# Set killbill as the default user
#USER killbill
USER 10001
WORKDIR $KILLBILL_HOME
ENV TERM=xterm

RUN chmod -R 755 /var/lib/killbill && mkdir -p $KILLBILL_HOME/.m2

# Setup Maven
RUN mkdir -p $KILLBILL_HOME/.m2
COPY /docker/templates/build/settings.xml $KILLBILL_HOME/.m2/settings.xml

# Setup git
RUN git config --global user.name "Kill Bill core team" && \
    git config --global user.email "contact@killbill.io" && \
    git config --global push.default simple && \
    git config --global credential.helper store

CMD ["bash"]
