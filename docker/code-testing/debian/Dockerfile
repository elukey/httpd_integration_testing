ARG image_name

FROM ${image_name}

MAINTAINER Luca Toscano <toscano.luca@gmail.com>

# ARGs defined after the FROM are available to use in the
# rest of the Dockerfile.
# https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG branch='2.4.x'
ARG configure='--enable-mpms-shared --enable-maintainer-mode --enable-modules=reallyall --enable-load-all-module'
ARG image_name

ENV DEBIAN_FRONTEND=noninteractive

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
  wget \
  php-fpm

RUN bash -c 'if [ "$image_name" == "debian:stretch" ] ; then echo "deb http://deb.debian.org/debian stretch-backports main" >> /etc/apt/sources.list ; fi'

# Update the repository sources list
RUN apt-get update
RUN apt-get install --yes libbrotli-dev libbrotli1

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

USER nobody

# Checkout the httpd branch and the perl testing framework
RUN bash -c 'if [ "$branch" == "trunk" ] ; then svn checkout https://svn.apache.org/repos/asf/httpd/httpd/trunk /tmp/httpd-trunk; else svn checkout https://svn.apache.org/repos/asf/httpd/httpd/branches/${branch}/ /tmp/httpd-${branch}; fi'
RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-${branch}
RUN ./buildconf --with-apr=apr-1-config
RUN ./configure ${configure}
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
