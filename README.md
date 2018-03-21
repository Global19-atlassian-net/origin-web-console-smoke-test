# Docker image of Protractor with headless Chrome

## Run locally

This will run protractor against Chrome allowing you to see the tests:

```bash
yarn install
$(yarn bin)/webdriver-manager update
CONSOLE_URL=https://<machine-ip>:8443 $(yarn bin)/protractor protractor.conf.js
```

## Build the container

```bash
$ ./docker_build.sh
# or
$ docker build -t protractor-smoke-test .
```

# Run the tests in a container

```bash
$ CONSOLE_URL=https://<machine-ip>:8443 ./docker_run.sh
# or
$ docker run -it --privileged --rm --shm-size 2g -v $(pwd)/test:/protractor -e CONSOLE_URL=https://<machine-ip>:8443 protractor-smoke-test
```

# Debug from within the container

```bash
$ ./docker_debug.sh
# then, in the container:
# $ CONSOLE_URL=https://<machine-ip>:8443 protractor protractor.conf.js
# or
$ docker run -it --privileged --rm --shm-size 2g -v $(pwd)/test:/protractor -e CONSOLE_URL=https://<machine-ip>:8443 --entrypoint /bin/bash protractor-smoke-test
$ protractor protractor.conf.js
```

Based on [Docker Protractor Headless](https://github.com/jciolek/docker-protractor-headless)

## Collecting metrics

[Prom Client](https://github.com/siimon/prom-client) for Node.js is used to collect metrics.

There is a server.js file in this repository, it can be run with:

```bash
node server.js
```

This will start a server listening at `http://localhost:3000/metrics`.  When this endpoint
is hit, via a browser or otherwise, a txt file like the following will be returned:

```
# lots of default metrics...
# followed by:
#
# HELP origin_web_console_smoke_test The number of times the web console smoke tests pass (should be 1)
# TYPE origin_web_console_smoke_test counter
origin_web_console_smoke_test 2
```

This is implemented as a prometheus Counter.  It is a proof of concept only. Some things we need
to decide:

- how often should these smoke tests run? only on upgrades?  constantly?
- what is the most meaningful metric to gather?

In addition, there is probably a better way to stitch together `protractor` to the `/metrics`
endpoint.  Currently this just reads the JUNIT `xml` file & looks for `failures: 0` on
each of the `testsuite` entries.  We could write a custom reporter to simplify this &
eliminate the need to read the XML file.
