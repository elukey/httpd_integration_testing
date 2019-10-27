ARG image_name

FROM ${image_name}

MAINTAINER Luca Toscano <toscano.luca@gmail.com>

# ARGs defined after the FROM are available to use in the
# rest of the Dockerfile.
# https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG configure_args='--enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --enable-modules=reallyall --enable-load-all-module'
ARG httpd_version
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
  check \
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
  Protocol::HTTP2::Client \
  AnyEvent \
  FCGI

USER nobody

RUN mkdir /tmp/httpd_release_candidate
WORKDIR /tmp/httpd_release_candidate

# Get the release given as arg, check its sha1sum and unpack it
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz.sha256
RUN sha256sum --check httpd-${httpd_version}.tar.gz.sha256
RUN tar xvfz httpd-${httpd_version}.tar.gz

WORKDIR /tmp/httpd_release_candidate/httpd-${httpd_version}

RUN ./configure ${configure_args}
RUN make

USER root

RUN make install

# Temporary workaround to have php-fpm in the user's PATH
RUN ln -s $(ls /usr/sbin/php-fpm7*) /usr/local/bin/php-fpm

USER nobody

WORKDIR /tmp/httpd-test

RUN perl Makefile.PL -apxs /usr/local/apache2/bin/apxs

WORKDIR /tmp
