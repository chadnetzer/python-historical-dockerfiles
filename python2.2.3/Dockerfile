# Adapted from https://github.com/docker-library/python
FROM alpine:latest as builder

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN apk add --no-cache ca-certificates

ENV GPG_KEY 5B8F9EDBFD71B297
ENV PYTHON_VERSION 2.2.3

RUN set -ex \
	&& apk update \
	&& apk add --no-cache --virtual .fetch-deps \
		gnupg \
		tar \
		rpm \
	\
	# Extract from src.rpm which is signed (whereas the pre-Python2.3 tarballs are not)
	&& wget -O python.src.rpm "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/rpms/python-$PYTHON_VERSION-26.src.rpm" \
	#&& wget -O KRUD-GPG-KEY "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/rpms/KRUD-GPG-KEY" \
	# Use key downloaded from different location than tarball, just to be paranoid
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	&& gpg --armor --export $GPG_KEY >KRUD-GPG-KEY \
	&& rpm --import KRUD-GPG-KEY \
	&& rpm -K python.src.rpm \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" KRUD-GPG-KEY \
	&& rpm2cpio python.src.rpm | cpio -idmv "Python-$PYTHON_VERSION.tgz" \
	&& mkdir -p /usr/src/python \
	&& tar -xzC /usr/src/python --strip-components=1 -f Python-$PYTHON_VERSION.tgz \
	&& rm Python-$PYTHON_VERSION.tgz \
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

COPY posix_close_py223.patch /tmp/posix_close.patch
COPY no_nis_py223.patch /tmp/no_nis.patch
RUN set -ex \
	&& cd /usr/src/python \
	&& patch -p1 < /tmp/posix_close.patch \
	&& patch -p1 < /tmp/no_nis.patch \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-shared \
		--enable-unicode=ucs4 \
		--with-system-expat \
		--with-system-ffi \
		--without-cxx \
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
	&& apk del .build-deps \
	\
	&& find /usr/local -depth \
		\( \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' + \
	\
	&& python -V


FROM alpine:latest

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# install ca-certificates so that HTTPS works consistently
# other runtime dependencies for Python are installed later
RUN apk add --no-cache ca-certificates

COPY --from=builder /usr/local /usr/local/

CMD ["python"]
