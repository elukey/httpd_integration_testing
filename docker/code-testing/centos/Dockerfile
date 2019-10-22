ARG image_name
ARG trunk_configure_args=--enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --with-test-suite=/tmp/httpd-test --enable-modules=reallyall --enable-load-all-module
ARG 24x_configure_args=--enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --enable-modules=reallyall --enable-load-all-module

FROM ${image_name}

ARG httpd_version

MAINTAINER Luca Toscano <toscano.luca@gmail.com>

# Update the repository sources list
RUN yum update -y

# Install compiler and perl deps
RUN yum install -y \
 make automake gcc gcc-c++ \
 automake \
 perl \
 expat-devel \
 zlib-devel \
 ca-certificates

# Install openssl libs/deps
RUN yum install -y \
  openssl \
  openssl-devel

# Install cpanminus and perl dependencies
RUN cpan -i App::cpanminus
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
  FCGI

# Install dependencies to run the perl test suite and the mod_h2 one
RUN yum install -y apr \
  apr-devel \
  apr-util-devel \
  subversion \
  libtool \
  pcre-devel \
  pcre \
  lua-devel \
  lua \
  lua-libs \
  libnghttp2 \
  check \
  git \
  python3 \
  python3-pytest \
  python3-requests \
  rsync \
  curl \
  vim \
  wget \
  php-fpm \
  brotli

USER nobody

# Checkout httpd trunk and the perl testing framework
RUN svn checkout https://svn.apache.org/repos/asf/httpd/httpd/trunk /tmp/httpd-trunk
RUN svn checkout https://svn.apache.org/repos/asf/httpd/httpd/branches/2.4.x/ /tmp/httpd-2.4.x
RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-trunk

# Configure and build httpd trunk
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure ${trunk_configure_args}
RUN make

WORKDIR /tmp/httpd-2.4.x
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure ${24x_configure_args}
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

WORKDIR /tmp/httpd-test

RUN perl Makefile.PL -apxs /usr/local/apache2/bin/apxs

WORKDIR /tmp