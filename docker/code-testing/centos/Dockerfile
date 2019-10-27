ARG image_name

FROM ${image_name}

# ARGs defined after the FROM are available to use in the
# rest of the Dockerfile.
# https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG branch='2.4.x'
ARG configure='--enable-mpms-shared --enable-maintainer-mode --enable-modules=reallyall --enable-load-all-module'
ARG image_name

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

# lua-devel package still not available on centos8
# https://bugzilla.redhat.com/show_bug.cgi?id=1734218
# ===========================================================
RUN yum install -y gcc \
  gcc-c++ \
  kernel-devel \
  readline-devel \
  patch

COPY lua-5.3.5-shared_library-1.patch /tmp/lua_build/lua-5.3.5-shared_library-1.patch

RUN bash -c 'if [ "$image_name" == "centos:centos8" ] ; then cd /tmp/lua_build; curl -R -O https://www.lua.org/ftp/lua-5.3.4.tar.gz; tar -zxf lua-5.3.4.tar.gz; cd lua-5.3.4; patch -p1 < /tmp/lua_build/lua-5.3.5-shared_library-1.patch; make linux; make install; fi'

RUN bash -c 'if [ "$image_name" != "centos:centos8" ] ; then apt-get install -y lua-devel lua-libs; fi'
# ===========================================================


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