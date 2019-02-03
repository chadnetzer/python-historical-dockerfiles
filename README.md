## Dockerfiles for older Python versions

These are some Dockerfiles I'm writing to easily run older Python versions.  My main goal is to make it easier to demonstrate older Python behaviors when tutoring or answering questions for learners.  It is *not* my goal to produce containers for any sort of production usage, or for running older Python programs, etc.  **Please** do not rely on the containers produced here for anything other than educational tinkering. :)

Some of the things these containers can demonstrate are:

* Older split between ints and longs
 * `(sys.maxint + 1)`  -->  OverflowError in pre-v2.2
 * `1 << 63`  -->  -9223372036854775808 in prev2.4 (64 bit systems)
 * `10**100`  -->  0 in pre-v2.5
* None being assignable (SyntaxWarning in python2.3, but assignable)
* bool types being assignable (pre-Python 3), or non-existent (pre-v2.2)
* Older string module functions (pre-v2.2)
* Pre-iterator file line looping.
* And probably more that I'll think of later...


## Building

`cd` to a directory with a Dockerfile and run docker build, ie.:

```
cd python2.3
docker build -t $(basename $PWD) .
```

### Usage

To start built image interactively and to have the container removed after the session is finished, do something like:

```
$ docker run -it --rm python2.3
```

For more information, you can look at the official DockerHub Python pages for ideas on how the up-to-date releases can be used.

https://hub.docker.com/_/python/

## TODO

* Getting the interpreter history buffer and editing (ie. readline) to work on versions pre-v2.4

## License

These Dockerfiles are released under the MIT license.  They were based on the MIT licensed Alpine build of the Docker Python 2.7.15 image.
