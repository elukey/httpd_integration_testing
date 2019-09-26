# httpd integration testing tests

In this repository there are some prototypes of possible solutions to automate the integration testing of the httpd code.

## Docker

The Docker images can be build and executed like the following:

- `docker build . -t $name-of-the-image`
- `docker run $name-of-the-image $command`

The idea for the Docker images is to checkout the httpd-trunk, httpd-framework and mod_h2 repos in the same place, building and configuring them to be ready to use.

To run the perl test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/httpd-trunk && make check`

To run the mod_h2 test suite: `docker run $name-of-the-image /bin/bash -c "cd /tmp/mod_h2 && make test`

Note: the mod_h2's test suite requires python 3.7 so it may not run on all platforms.

### Debian and Ubuntu

Debian and Ubuntu share the same docker image, that must be built with `--build_arg image_name=$docker_hub_image_name` with the following possible values:

* `ubuntu`
* `debian:jessie`
* `debian:stretch`
* `debian:buster`
*  etc..
