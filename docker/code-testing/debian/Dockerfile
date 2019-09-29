ARG image_name

FROM ${image_name}

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
  LWP::Protocol::https \
  FCGI \
  brotli

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
  php-fpm

USER nobody

# Checkout httpd trunk and the perl testing framework
RUN svn checkout https://svn.apache.org/repos/asf/httpd/httpd/trunk /tmp/httpd-trunk
RUN svn checkout https://svn.apache.org/repos/asf/httpd/httpd/branches/2.4.x/ /tmp/httpd-2.4.x
RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-trunk

# Configure and build httpd trunk
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure --enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --with-test-suite=/tmp/httpd-test --enable-modules=reallyall --enable-load-all-module
RUN make

WORKDIR /tmp/httpd-2.4.x
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure --enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --with-test-suite=/tmp/httpd-test --enable-modules=reallyall --enable-load-all-module
RUN make

USER root

RUN make install

# Temporary workaround to have php-fpm in the user's PATH
RUN ln -s $(ls /usr/sbin/php-fpm7*) /usr/local/bin/php-fpm

USER nobody

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
