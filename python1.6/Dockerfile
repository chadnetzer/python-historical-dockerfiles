# Adapted from https://github.com/docker-library/python
FROM alpine:latest as builder

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# These older releases aren't signed, but have published MD5 sums on the
# python.org website.  I have taken this MD5 sum from:
# https://www.python.org/download/releases/2.0.1/
ENV MD5_SUM 9d72ef93d7698769d9d3be7c17d5ad92
ENV PYTHON_VERSION 1.6

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
	&& apk add --no-cache --virtual .fetch-deps \
		tar \
	\
	&& wget -O python.tgz "https://www.python.org/ftp/python/src/python-$PYTHON_VERSION.tar.gz" \
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
		readline \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		zlib-dev

COPY posix_close_py16.patch /tmp/posix_close.patch
RUN set -ex \
	&& cd /usr/src/python \
	&& patch -p1 < /tmp/posix_close.patch \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-shared \
		--enable-unicode=ucs4 \
		--with-system-expat \
		--with-system-ffi \
		--without-cxx \
	&& make \
# set thread stack size to 1MB so we don't segfault before we hit sys.getrecursionlimit()
# https://github.com/alpinelinux/aports/commit/2026e1259422d4e0cf92391ca2d3844356c649d0
		EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
	&& make install \
	\
	&& find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec scanelf --needed --nobanner --format '%n#p' '{}' ';' \
		| tr ',' '\n' \
		| sort -u \
		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
		| xargs -rt apk add --no-cache --virtual .python-rundeps \
	&& find /usr/local -depth \
		\( \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' + \
	\
	&& python -c "import sys; print sys.version"


FROM alpine:latest

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
		readline

COPY --from=builder /usr/local /usr/local/

CMD ["python"]
