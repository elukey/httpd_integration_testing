ARG image_name
ARG configure_args='--enable-mpms-shared --enable-maintainer-mode --enable-modules=reallyall --enable-load-all-module'

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
  Protocol::HTTP2::Client \
  AnyEvent \
  FCGI

# Install dependencies to run the perl test suite
RUN yum install -y apr \
  apr-devel \
  apr-util-devel \
  subversion \
  libtool \
  pcre-devel \
  pcre \
  libnghttp2 \
  check \
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

WORKDIR /tmp

RUN svn checkout https://svn.apache.org/repos/asf/httpd/test/framework/trunk /tmp/httpd-test

WORKDIR /tmp/httpd-test

RUN perl Makefile.PL -apxs /usr/local/apache2/bin/apxs

WORKDIR /tmp
