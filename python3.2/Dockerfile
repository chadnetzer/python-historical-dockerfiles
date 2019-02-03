# Adapted from https://github.com/docker-library/python
FROM alpine:latest as builder

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

ENV GPG_KEY 26DEA9D4613391EF3E25C9FF0A5B101836580288
ENV PYTHON_VERSION 3.2.6

COPY pgp_keys_36580288.asc /tmp/pgp_keys.asc

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
	&& apk add --no-cache --virtual .fetch-deps \
		gnupg \
		tar \
		xz \
	\
	&& wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	#&&  (gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	#  || gpg --batch --keyserver pgp.mit.edu --recv-keys "$GPG_KEY" \
	#  || gpg --batch --keyserver keyserver.pgp.com --recv-keys "$GPG_KEY" ) \
	&& gpg --batch --import /tmp/pgp_keys.asc \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm python.tar.xz \
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

COPY posix_close_py32.patch /tmp/posix_close.patch

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
	&& make -j "$(nproc)" \
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
	&& python3 --version


FROM alpine:latest

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
		readline

COPY --from=builder /usr/local /usr/local/

CMD ["python3"]
