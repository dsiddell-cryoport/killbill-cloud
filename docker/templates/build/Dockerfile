FROM ubuntu:20.04
LABEL maintainer="killbilling-users@googlegroups.com"

USER root

ENV LC_CTYPE=en_US.UTF-8
ENV LC_ALL=C
ENV LANG=en_US.UTF-8
ENV PYTHONIOENCODING=utf8
ENV TRIVY_DISABLE_VEX_NOTICE=true

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
RUN mkdir -p $KILLBILL_HOME/.m2

#RUN adduser killbill \
#            --home $KILLBILL_HOME \
#            --disabled-password \
#            --gecos '' && \
#    usermod -aG sudo killbill && \
#    echo "killbill:killbill" | chpasswd && \
#    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

#RUN adduser 10001 \
#            --home $KILLBILL_HOME \
#            --disabled-password \
#            --gecos '' && \
#            --force-badname  || true && \
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
#RUN chmod -R 755 /var/lib/killbill && mkdir -p $KILLBILL_HOME/.m2

# Install RVM
RUN apt-get update && \
    apt-get install --no-install-recommends -y rvm && \
    rm -rf /var/lib/apt/lists/*

# Install Ruby
RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm install 3.0.0"

# Install dependencies
RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem install bundler -v '>=2.0.0'"
RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem install openssl -v '>=2.1.2'"

# Clear gem cache, uninstall stringio, and install the correct version
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem clean"
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem uninstall stringio -a -x"
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem install stringio -v '3.1.2' --force"

# Verify gem installation
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && gem list | grep stringio"

# Create a Gemfile dynamically
RUN echo "source 'https://rubygems.org'" > $KILLBILL_HOME/Gemfile && \
    echo "gem 'bundler', '>=2.0.0'" >> $KILLBILL_HOME/Gemfile && \
    echo "gem 'openssl', '>=2.1.2'" >> $KILLBILL_HOME/Gemfile #&& \
   # echo "gem 'stringio', '3.1.2'" >> $KILLBILL_HOME/Gemfile
    
# Add extra rubies
#RUN /bin/bash -l -c "rvm install 2.2.2 && rvm install 2.4.2 && rvm install jruby-9.1.14.0"

# Install Ruby and bundler
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm install 3.0.0"
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 2.7.0 && gem install bundler -v '1.17.3'"
#RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 2.7.0 && gem install openssl -v '>=2.1.2'"

# Copy Gemfile and Gemfile.lock into the working directory
#COPY Gemfile Gemfile.lock $KILLBILL_HOME/
WORKDIR $KILLBILL_HOME
RUN /bin/bash -c "source /usr/share/rvm/scripts/rvm && rvm use 3.0.0 && bundle install"


# Set killbill as the default user
#USER killbill
USER 10001
WORKDIR $KILLBILL_HOME
ENV TERM=xterm

# Setup Maven
#RUN mkdir -p $KILLBILL_HOME/.m2
COPY /docker/templates/build/settings.xml $KILLBILL_HOME/.m2/settings.xml

# Setup git
#RUN git config --global user.name "Kill Bill core team" && \
#    git config --global user.email "contact@killbill.io" && \
#    git config --global push.default simple && \
#    git config --global credential.helper store

CMD ["bash"]
