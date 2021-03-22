## Dockerfiles for older Python versions

These are some Dockerfiles I'm writing to easily run older Python versions.  My main goal is to make it easier to demonstrate older Python behaviors when tutoring or answering questions for learners.  It is *not* my goal to produce containers for any sort of production usage, or for running older Python programs, or even scripts, etc.

**Please** do not rely on the containers produced here for anything other than educational tinkering and nostalgia. :)

Some of the things these containers can demonstrate are:

* Older split between ints and longs
    * `(sys.maxint + 1)`  -->  OverflowError in pre-v2.2
    * `1 << 63`  -->  -9223372036854775808 in prev2.4 (64 bit systems)
    * `10**100`  -->  0 in pre-v2.5
* None being assignable (SyntaxWarning in python2.3, but assignable)
* bool types being assignable (pre-Python 3), or non-existent (pre-v2.2)
* Older string module functions (pre-v2.2)
* Pre-iterator file line looping.
* Only single-quoted string constants allowed (pre-v1.0)
* And probably more that I'll think of later...

This original project has grown to include "newer" versions of Python as well,
including those that are also in Dockerhub (such as Python 2.7, and Python
3.2+).  You should absolutely use those docker images instead of these if/when
they are available.  I mainly added support more recent versions for my own
personal testing, and am not supporting them for anything more than tinkering,
alpha-release testing, an so on.

### Building

`cd` to a directory with a Dockerfile (for example, python2.3), and run docker build, ie.:

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


#### Contributing/Suggestions

Feel free to open issues if you find problems with standard modules not working, or have any ideas for things you'd like to make better.

#### TODO

* Getting the interpreter history buffer and editing (ie. readline) to work on versions pre-v2.4

#### License

These Dockerfiles are released under the MIT license.  They were based on the MIT licensed Alpine build of the Docker Python 2.7.15 image.
