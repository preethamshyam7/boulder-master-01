Boulder - An ACME CA
====================

Quickstart
------

Boulder has a Dockerfile to make it easy to install and set up all its
dependencies. This is how the maintainers work on Boulder, and is our main
recommended way to run it.

Make sure you have a local copy of this Boulder in your `$GOPATH`:

    export GOPATH=~/gopath
    git clone https://github.com/preethamshyam7/boulder-prov-modified/ $GOPATH/src/github.com/preethamshyam7/boulder-prov-modified

To start Boulder in a Docker container, run:

    docker-compose up

To run tests:

    docker-compose run boulder ./test.sh

To run a specific unittest:

    docker-compose run boulder go test ./ra

The configuration in docker-compose.yml mounts your
[`$GOPATH`](https://golang.org/doc/code.html#GOPATH) on top of its own
`$GOPATH`. So you can edit code on your host and it will be immediately
reflected inside Docker images run with docker-compose.

If docker-compose fails with an error message like "Cannot start service
boulder: oci runtime error: no such file or directory" or "Cannot create
container for service boulder" you should double check that your `$GOPATH`
exists and doesn't contain any characters other than letters, numbers, `-`
and `_`.

If you have problems with Docker, you may want to try [removing all containers
and volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes).

By default, Boulder uses a fake DNS resolver that resolves all hostnames to
127.0.0.1. This is suitable for running integration tests inside the Docker
container. If you want Boulder to be able to communicate with a client running
on your host instead, you should find your host's Docker IP with:

    ifconfig docker0 | grep "inet addr:" | cut -d: -f2 | awk '{ print $1}'

And edit docker-compose.yml to change the FAKE_DNS environment variable to
match.

Alternatively, you can override the docker-compose.yml default with an environmental variable using -e (replace 172.17.0.1 with the host IPv4 address found in the command above)

    docker-compose run -e FAKE_DNS=172.17.0.1 --service-ports boulder ./start.py

Boulder's default VA configuration (`test/config/va.json`) is configured to
connect to port 5002 to validate HTTP-01 challenges and port 5001 to validate
TLS-SNI-01 challenges. If you want to solve challenges with a client running on
your host you should make sure it uses these ports to respond to validation
requests, or update the VA configuration's `portConfig` to use ports 80 and 443
to match how the VA operates in production and staging environments. If you use
a host-based firewall (e.g. `ufw` or `iptables`) make sure you allow connections
from the Docker instance to your host on the required ports.

If a base image changes you will need to rebuild
images for both the boulder and bhsm containers and re-create them. The quickest way
to do this is with this command:

    ./docker-rebuild.sh

