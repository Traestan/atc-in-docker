FROM ubuntu:14.04

# Enable the Ubuntu multiverse repository.
RUN echo "deb http://us.archive.ubuntu.com/ubuntu/ trusty multiverse" >> /etc/apt/source.list
RUN echo "deb-src http://us.archive.ubuntu.com/ubuntu/ trusty multiverse">> /etc/apt/source.list
RUN echo "deb http://us.archive.ubuntu.com/ubuntu/ trusty-updates multiverse" >> /etc/apt/source.list
RUN echo "deb-src http://us.archive.ubuntu.com/ubuntu/ trusty-updates multiverse" >> /etc/apt/source.list

# Install Dependencies.
RUN apt-get update && apt-get install -y autoconf automake bison build-essential fail2ban gawk git-core groff \
    groff-base erlang-dev libasound2-dev libavformat-dev libdb-dev libexpat1-dev libcurl4-openssl-dev \
    libgdbm-dev libgnutls-dev libjpeg-dev libmp3lame-dev libncurses5 libncurses5-dev libperl-dev \
    libogg-dev libsnmp-dev libssl-dev libtiff4-dev libtool libvorbis-dev libx11-dev libzrtpcpp-dev \
    make portaudio19-dev python-dev snmp snmpd subversion unixodbc-dev uuid-dev zlib1g-dev libsqlite3-dev \
    libpcre3-dev libspeex-dev libspeexdsp-dev libldns-dev libedit-dev libladspa-ocaml-dev libmemcached-dev  \
    libmp4v2-dev libmyodbc libpq-dev libvlc-dev libv8-dev liblua5.2-dev libyaml-dev libpython-dev odbc-postgresql \
    sendmail unixodbc wget yasm

# Use Gawk.
RUN update-alternatives --set awk /usr/bin/gawk

# Install source code dependencies.
ADD deps/install.sh /root/install-deps.sh
WORKDIR /root
RUN chmod +x install-deps.sh
RUN ./install-deps.sh
RUN rm install-deps.sh

# Configure Fail2ban
ADD conf/freeswitch.conf /etc/fail2ban/filter.d/freeswitch.conf
ADD conf/freeswitch-dos.conf /etc/fail2ban/filter.d/freeswitch-dos.conf
ADD conf/f2b.local /etc/fail2ban/f2b.local

# Download FreeSWITCH.
WORKDIR /usr/src
ENV GIT_SSL_NO_VERIFY=1
RUN git clone https://freeswitch.org/stash/scm/fs/freeswitch.git -b v1.6.5

# Bootstrap the build.
WORKDIR freeswitch
RUN ./bootstrap.sh

# Enable the desired modules.
ADD modules/modules.conf /usr/src/freeswitch/modules.conf

# Build FreeSWITCH.
RUN ./configure --enable-core-pgsql-support
RUN make
RUN make install
RUN make uhd-sounds-install
RUN make uhd-moh-install
RUN make samples

# Post install configuration.
ADD sys/init /etc/init.d/freeswitch
RUN chmod +x /etc/init.d/freeswitch
RUN update-rc.d -f freeswitch defaults
ADD sys/default /etc/default/freeswitch

# Add the freeswitch user.
RUN adduser --gecos "FreeSWITCH Voice Platform" --no-create-home --disabled-login --disabled-password --system --ingroup daemon --home /usr/local/freeswitch freeswitch
RUN chown -R freeswitch:dazemon /usr/local/freeswitch

# Create the log file.
RUN touch /usr/local/freeswitch/log/freeswitch.log
RUN chown freeswitch:daemon /usr/local/freeswitch/log/freeswitch.log

# Open the container up to the world.
EXPOSE 5060/tcp 5060/udp 5080/tcp 5080/udp
EXPOSE 5066/tcp 7443/tcp
EXPOSE 8021/tcp
EXPOSE 64535-65535/udp

# Start the container.
CMD service snmpd start && service freeswitch start && tail -f /usr/local/freeswitch/log/freeswitch.log