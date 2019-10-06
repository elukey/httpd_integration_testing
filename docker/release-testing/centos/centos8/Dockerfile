ARG image_name

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
RUN yum install -y cpanminus
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
  #lua-devel \
  #lua \
  #lua-libs \
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


# lua-devel package still not available
# https://bugzilla.redhat.com/show_bug.cgi?id=1734218
RUN yum install -y gcc \
  gcc-c++ \
  kernel-devel \
  readline-devel \
  patch

USER nobody
RUN mkdir /tmp/lua_build
WORKDIR /tmp/lua_build
RUN wget http://www.linuxfromscratch.org/patches/blfs/9.0/lua-5.3.5-shared_library-1.patch
RUN curl -R -O http://www.lua.org/ftp/lua-5.3.4.tar.gz
RUN tar -zxf lua-5.3.4.tar.gz
WORKDIR lua-5.3.4
RUN patch -p1 < ../lua-5.3.5-shared_library-1.patch
RUN make linux

USER root
RUN make install

USER nobody

RUN mkdir /tmp/httpd_release_candidate
WORKDIR /tmp/httpd_release_candidate

# Get the release given as arg, check its sha1sum and unpack it
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz
RUN wget https://dist.apache.org/repos/dist/dev/httpd/httpd-${httpd_version}.tar.gz.sha256
RUN sha256sum --check httpd-${httpd_version}.tar.gz.sha256
RUN tar xvfz httpd-${httpd_version}.tar.gz

WORKDIR /tmp/httpd_release_candidate/httpd-${httpd_version}

RUN ./configure --enable-mpms-shared --enable-maintainer-mode --with-crypto --with-openssl --enable-lua --with-test-suite=/tmp/httpd-test --enable-modules=reallyall --enable-load-all-module
RUN make

USER root

RUN make install

# Temporary workaround to have php-fpm in the user's PATH
RUN ln -s $(ls /usr/sbin/php-fpm7*) /usr/local/bin/php-fpm

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

WORKDIR /tmp/httpd-test

RUN perl Makefile.PL -apxs /usr/local/apache2/bin/apxs

# Fix t/filter/case.t - see https://bz.apache.org/bugzilla/show_bug.cgi?id=63787
USER root
RUN for version in $(ls /usr/share/perl); do touch /usr/share/perl/$version/pod/perlsub.pod; done

WORKDIR /tmp
