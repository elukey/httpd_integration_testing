# httpd integration testing

In this repository there are some prototypes of possible solutions to automate the integration testing of the httpd code.

## Docker

The Docker images can be build and executed like the following:

- `docker build . --build-arg X=foo --build-arg X2=bar .. -t $name-of-the-image`
- `docker run $name-of-the-image $command`

The idea for the Docker images is to checkout a httpd branch and the httpd-framework repos in the same place, building and configuring them to be ready to use.

### Debian and Ubuntu

Debian and Ubuntu share the same docker image, that must be built with `--build_arg image_name=$docker_hub_image_name` with the following possible values:

* `ubuntu`
* `debian:jessie`
* `debian:stretch`
* `debian:buster`
*  etc..

### CentOS

CentOS 8 doesn't ship with the `lua-devel` package, so it needs to be build manually in the Dockerfile.

### Release candidate testing

These docker images focus on getting a certain release candidate tarball, unpack/check/configure/build it and allow to run the Perl test suite.

To build the image (example): `docker build --build-arg image_name=debian:buster --build-arg httpd_version='2.4.40' --buil-arg trunk_configure_args='--enable-mpms-shared --enable-maintainer-mode ..' --buil-arg=24x_configure_args='--enable-mpms-shared --enable-maintainer-mode ..' . -t debian/buster/httpd-tests-ga`

To run the perl test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-test && ./t/TEST"`

### Code snapshot testing

These docker images focus on getting the latest version of the trunk and 2.x.x branches, configure/build them and allow to run the Perl test suite.

To build the image (example): `docker build --build-arg image_name=debian:buster --branch trunk --buil-arg configure='--enable-mpms-shared --enable-maintainer-mode ..' . -t debian/buster/httpd-tests-code-snapshot`

To run the perl test suite (trunk only, if `--with-test-suite=/tmp/httpd-test` is passed to the `configure` arg): `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-trunk && make check`

To run the perl test suite (2.4.x): `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-test && ./t/TEST"`

## TODOs:

* Add travis configuration.
* Find a way to improve reuse of configurations in Dockerfiles (at the moment there is a lot of repetition).
* Add support for more OSes.
