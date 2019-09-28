ARG image_name

FROM ${image_name}

ARG httpd_version

MAINTAINER Luca Toscano <toscano.luca@gmail.com>

# Update the repository sources list
RUN apt-get update

# Install compiler and perl deps
RUN apt-get install --yes \
 build-essential \
 automake \
 gcc-multilib \
 apt-utils \
 perl \
 expat \
 libexpat-dev \
 zlib1g-dev \
 ca-certificates

# Install openssl libs/deps
RUN apt-get install --yes \
  openssl \
  libssl-dev

# Install cpanminus and perl dependencies
RUN apt-get install -y cpanminus
RUN cpanm --force Bundle::ApacheTest \
  HTTP::DAV \
  DateTime \
  Time::HiRes \
  Protocol::HTTP2::Client \
  AnyEvent \
  Test \
  Test::Harness \
  Crypt::SSLeay \
  Net::SSLeay \
  IO::Socket::SSL \
  IO::Socket::IP \
  IO::Select \
  LWP::Protocol::https

# Install dependencies to run the perl test suite and the mod_h2 one
RUN apt-get install --yes libapr1 \
  libapr1-dev \
  libaprutil1-dev \
  subversion \
  libtool \
  libpcre3-dev \
  liblua5.3-dev \
  libnghttp2-dev \
  nghttp2 \
  check \
  git \
  python3 \
  python3-pytest \
  python3-requests \
  rsync \
  curl \
  vim \
  wget

USER nobody

RUN mkdir /tmp/httpd_release_candidate
WORKDIR /tmp/httpd_release_candidate

# Get the release given as arg, check its sha1sum and unpack it
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz.sha256
RUN sha256sum --check httpd-${httpd_version}.tar.gz.sha256
RUN tar xvfz httpd-${httpd_version}.tar.gz

WORKDIR /tmp/httpd_release_candidate/httpd-${httpd_version}

RUN ./configure --enable-mpms-shared --enable-maintainer-mode --enable-ssl --with-crypto --with-openssl --enable-lua --enable-deflate --with-test-suite=/tmp/httpd-test --enable-http2
RUN make

USER root

RUN make install

USER nobody

# Checkout the mod_h2 github repo to checkout/configure its integration testing suite

WORKDIR /tmp

RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-test

RUN perl Makefile.PL -apxs /usr/local/apache2/bin/apxs

# Checkout the mod_h2 github repo to checkout/configure its integration testing suite

WORKDIR /tmp

RUN git clone https://github.com/icing/mod_h2.git

WORKDIR /tmp/mod_h2

RUN autoreconf -i

# Needed since the USER bash variable is not populated
# in Docker containers, and it is needed by the configure
# step below.
ENV USER=nobody

RUN ./configure -with-apxs=/usr/local/apache2/bin/apxs

RUN make

WORKDIR /tmp