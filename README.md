# httpd integration testing

In this repository there are some prototypes of possible solutions to automate the integration testing of the httpd code.

## Docker

The Docker images can be build and executed like the following:

- `docker build . --build-arg X=foo --build-arg X2=bar .. -t $name-of-the-image`
- `docker run $name-of-the-image $command`

The idea for the Docker images is to checkout the httpd-trunk, httpd-framework and mod_h2 repos in the same place, building and configuring them to be ready to use.

### Debian and Ubuntu

Debian and Ubuntu share the same docker image, that must be built with `--build_arg image_name=$docker_hub_image_name` with the following possible values:

* `ubuntu`
* `debian:jessie`
* `debian:stretch`
* `debian:buster`
*  etc..

### Release candidate testing

These docker images focus on getting a certain release candidate tarball, unpack/check/configure/build it and allow to run the Perl or HTTP/2 test suite.

To run the perl test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-test && ./t/TEST"`

To run the mod_h2 test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/mod_h2 && make test"`

Note: the mod_h2's test suite requires python 3.5+ so it may not run on all platforms.

### Code snapshot testing

These docker images focus on getting the latest version of the trunk and 2.4.x branches, configure/build them and allow to run the Perl or HTTP/2 test suite.

To run the perl test suite (trunk only): `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-trunk && make check`

To run the perl test suite (2.4.x): `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-test && ./t/TEST"`

To run the mod_h2 test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/mod_h2 && make test"`

Note: the mod_h2's test suite requires python 3.5+ so it may not run on all platforms.

