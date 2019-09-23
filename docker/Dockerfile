FROM debian:buster

MAINTAINER Luca Toscano <toscano.luca@gmail.com>

# Update the repository sources list
RUN apt-get update

# Install compiler and perl deps
RUN apt-get install --yes \
 build-essential \
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

# Install dependencies to run the perl test suite
RUN apt-get install --yes libapr1 \
  libapr1-dev \
  libaprutil1-dev \
  subversion \
  libtool \
  libpcre3-dev \
  liblua5.3-dev \
  libnghttp2-dev \
  nghttp2 \
  check

USER nobody

# Checkout httpd trunk and the perl testing framework
RUN svn checkout https://svn.apache.org/repos/asf/httpd/httpd/trunk /tmp/httpd-trunk
RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-trunk

# Configure and build httpd trunk
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure --enable-maintainer-mode --enable-ssl --with-crypto --with-openssl --enable-lua --enable-deflate --with-test-suite=/tmp/httpd-test --enable-http2
RUN make
