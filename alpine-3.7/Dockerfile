FROM oondeo/alpine:3.7

ENV SUMMARY="Nginx and php image with standar modules"	\
    DESCRIPTION="The image use scripts and configurations compatible \
        with redhat openshift."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Nginx" \
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i \
      io.s2i.scripts-url=image:///usr/libexec/s2i \
      com.redhat.component="nginx" \
      name="oondeo/php-nginx" \
      version="7" \
      release="2" \
      maintainer="OONDEO <info@oondeo.es>"

USER root

# ensure local pypy is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# install ca-certificates so that HTTPS works consistently
# the other runtime dependencies for PyPy are installed later
RUN apk add --no-cache ca-certificates

ARG PYPY_VERSION=5.10.0
ARG PYPY_SHA256SUM=a6e4cffde71e3f08b6e1befa5c0352a9bcc5f4e9f5cbf395001e0763a1a0d9e3

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ARG PYTHON_PIP_VERSION=9.0.1

# BUILD
RUN apk add --no-cache --virtual .build-deps \
        bzip2-dev \
        expat-dev \
        gcc \
        gdbm-dev \
        libc-dev \
        libffi-dev \
        libressl-dev \
        linux-headers \
        make \
        ncurses-dev \
        pax-utils \
        readline-dev \
        sqlite-dev \
        tar \
        tk \
        tk-dev \
        xz-dev \
        zlib-dev \
        wget \
        py2-pip \
        python2-dev \
        libressl-dev \
    && pip install --no-cache-dir pycparser cffi \
    && cd /tmp \
    && wget -O pypy.tar.bz2 "https://bitbucket.org/pypy/pypy/downloads/pypy3-v${PYPY_VERSION}-src.tar.bz2" \
    && echo "$PYPY_SHA256SUM *pypy.tar.bz2" | sha256sum -c - \
    && mkdir -p /usr/src/pypy \
    && tar -xjC /usr/src/pypy --strip-components=1 -f pypy.tar.bz2 \
    && cd /usr/src/pypy/pypy/goal \
    && python ../../rpython/bin/rpython --opt=jit \
    && cp -a * /usr/local/ \
    && install-deps /usr/local \
    && wget -O /tmp/get-pip.py 'https://bootstrap.pypa.io/get-pip.py' \
    && /usr/local/bin/pypy3 /tmp/get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        "pip==$PYTHON_PIP_VERSION" \
    && pip3 uninstall pycparser cffi && apk del .build-deps \
    && rm -rf /tmp/* /usr/src/*

RUN /usr/bin/docker-footer

CMD ["pypy3"]

USER 1001