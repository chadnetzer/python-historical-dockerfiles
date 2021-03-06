# Adapted from https://github.com/docker-library/python
FROM alpine:latest as builder

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

ENV GPG_KEY DBBF2EEBF925FAADCF1F3FFFD9866941EA5BBD71
ENV PYTHON_VERSION 3.0.1

COPY barrypub-gpg.asc /tmp/pgp_keys.asc

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
	&& apk add --no-cache --virtual .fetch-deps \
		gnupg \
		tar \
		bzip2 \
	\
	&& wget -O python.tar.bz2 "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.bz2" \
	&& wget -O python.tar.bz2.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.bz2.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	#&&  (gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	#  || gpg --batch --keyserver pgp.mit.edu --recv-keys "$GPG_KEY" \
	#  || gpg --batch --keyserver keyserver.pgp.com --recv-keys "$GPG_KEY" ) \
	&& gpg --batch --import /tmp/pgp_keys.asc \
	&& gpg --batch --verify python.tar.bz2.asc python.tar.bz2 \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" python.tar.bz2.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xjC /usr/src/python --strip-components=1 -f python.tar.bz2 \
	&& rm python.tar.bz2 \
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
		patch \
		pax-utils \
		readline-dev \
		readline \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		zlib-dev

COPY posix_close_py30.patch /tmp/posix_close.patch

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
	&& python3.0 --version


FROM alpine:latest

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

RUN set -ex \
	&& apk update \
	&& apk add --no-cache ca-certificates \
		readline

COPY --from=builder /usr/local /usr/local/

CMD ["python3.0"]
