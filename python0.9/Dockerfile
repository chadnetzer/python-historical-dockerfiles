# Adapted from https://github.com/docker-library/python
FROM alpine:latest as builder

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN apk add --no-cache ca-certificates

# These older releases aren't signed, but have published MD5 sums on the
# python.org website.  I have taken this MD5 sum from:
# https://www.python.org/download/releases/2.0.1/
ENV MD5_SUM 7d7a4c0e472226b33f7ad2f976f4eb9e
ENV PYTHON_VERSION 0.9.1

RUN set -ex \
	&& apk update \
	&& apk add --no-cache --virtual .fetch-deps \
		tar \
	\
	&& wget -O python.tgz "https://www.python.org/ftp/python/src/Python-$PYTHON_VERSION.tar.gz" \
	&& echo "$MD5_SUM  python.tgz" >python.tgz.md5sum \
	&& md5sum -c python.tgz.md5sum \
	&& mkdir -p /usr/src/python \
	&& tar -xzC /usr/src/python --strip-components=1 -f python.tgz \
	&& rm python.tgz \
	\
	&& apk add --no-cache --virtual .build-deps  \
		bzip2-dev \
		coreutils \
		dpkg-dev dpkg \
		expat-dev \
		findutils \
		gcc \
		gdbm-dev \
		libc-dev \
		libffi-dev \
		libnsl-dev \
		libressl-dev \
		libtirpc-dev \
		linux-headers \
		make \
		ncurses-dev \
		pax-utils \
		readline-dev \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		zlib-dev \
# add build deps before removing fetch deps in case there's overlap
	&& apk del .fetch-deps

RUN set -ex \
	&& cd /usr/src/python/src \
	&& make \
# set thread stack size to 1MB so we don't segfault before we hit sys.getrecursionlimit()
# https://github.com/alpinelinux/aports/commit/2026e1259422d4e0cf92391ca2d3844356c649d0
		EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
	&& install python /usr/local/bin \
	&& cd ../lib \
	&& mkdir /usr/local/lib/python \
	&& install *.py /usr/local/lib/python \
	\
	&& apk del .build-deps


FROM alpine:latest

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN apk add --no-cache ca-certificates

COPY --from=builder /usr/local /usr/local/

CMD ["python"]
