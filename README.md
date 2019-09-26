# httpd integration testing tests

In this repository there are some prototypes of possible solutions to automate the integration testing of the httpd code.

## Docker

The Docker images can be build and executed like the following:

- `docker build .`
- `docker run $image_id $command`

To run the perl test suite, `$command` must be `make check`.

### Debian and Ubuntu

Debian and Ubuntu share the same docker image, that must be built with `--build_arg image_name=$docker_hub_image_name` with the following possible values:

* `ubuntu`
* `debian/jessie`
* `debian/stretch`
* `debian/buster`
*  etc..
