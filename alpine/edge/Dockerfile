FROM gliderlabs/alpine:edge

ENV LANG C.UTF-8

# Find versions here: https://www.python.org/downloads/source/
ONBUILD ARG PYTHON_VERSION

ONBUILD RUN set -ex \
    && if [ -z $PYTHON_VERSION ]; then echo "PYTHON_VERSION required"; exit 2; fi \
    && echo "trying python build=$PYTHON_VERSION" \
    && apk -q update \
    && apk -q upgrade \
    && apk -q add --virtual .build-deps \
        build-base \
        libc-dev \
        linux-headers \
        openssl-dev \
        libxml2-dev \
        libxslt-dev \
        sqlite-dev \
        zlib-dev \
        xz-dev \
        bzip2-dev \
        readline-dev \
        ncurses-dev \
        gdbm-dev \
        curl \
        ca-certificates \
        pax-utils \
        gpgme \
        gzip \
    && cd /tmp \
    && curl -fsSLO https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tgz \
    && curl -fsSLO https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tgz.asc \
    && curl -fsSL https://www.python.org/static/files/pubkeys.txt | gpg --homedir /tmp --import \
    && curl -fsSLO https://bootstrap.pypa.io/get-pip.py \
    && gpg --homedir /tmp --verify Python-$PYTHON_VERSION.tgz.asc \
    && tar xzf Python-$PYTHON_VERSION.tgz \
    && cd Python-$PYTHON_VERSION \
    && CFLAGS="-Os -pipe" \
        LDFLAGS="-Os -pipe" \
        ./configure \
        --enable-shared \
        --enable-loadable-sqlite-extensions \
    && make -s -j$(getconf _NPROCESSORS_ONLN) \
    && make -s install \
    && SUFFIX=${PYTHON_VERSION%.[a-z0-9]*} \
    && if [ "$SUFFIX" = "2.4*" ] || [ "$SUFFIX" = "2.5*" ]; then \
          curl -fsSLO https://raw.githubusercontent.com/pypa/setuptools/bootstrap-py24/ez_setup.py; \
        elif [ "${PYTHON_VERSION%%.[a-z0-9]*}" -le "2" ] ; then \
          echo "Skipping ez_setup.py; Incompatible python version $PYTHON_VERSION"; \
        else \
          curl -fsSLO https://bootstrap.pypa.io/ez_setup.py; \
        fi \
    && cd /usr/local/bin \
    # symlink binaries
    && if [ ! -f python-config ] && [ -f python$SUFFIX-config ]; then ln -s python$SUFFIX-config python-config; fi \
    && if [ ! -f python ] && [ -f python$SUFFIX ]; then ln -s python$SUFFIX python; fi\
    # install pip if not bundled with python
    && if [ ! -f pip ]; then python /tmp/get-pip.py; fi \
    && if [ ! -f pip ] && [ -f pip$SUFFIX ]; then ln -s pip$SUFFIX pip; fi \
    && if [ ! -f easy_install ] && [ -f easy_install-$SUFFIX ]; then ln -s easy_install-$SUFFIX easy_install; fi\
    # ensure pip is up to date
    && pip install --upgrade pip \
    # install setuptools if compatible
    && if [ -f /tmp/ez_setup.py ]; then python /tmp/ez_setup.py; fi \
    && python_libs="$( \
        scanelf -nBR /usr/local \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk -q add --virtual .python-libs $python_libs \
    && apk -q del -r .build-deps \
    && rm -rf /tmp/* /var/cache/apk/* /usr/local/share/* ~/.cache \
    && find /usr/local -type d -depth -name 'tests' -exec rm -rf {} + \
    && find /usr/local -type d -depth -name __pycache__ -exec rm -rf {} + \
    && find /usr/local -type f -depth -name '*.pyc' -exec rm -rf {} + \
    && find /usr/local -type f -depth -name '*.pyo' -exec rm -rf {} + \
    && find /usr/local -depth -name 'pydoc*' -exec rm -rf {} + \
    && find /usr/local -depth -name 'idle*' -exec rm -rf {} +
