FROM debian:stretch
LABEL maintainer="Odoo S.A. <info@odoo.com>"

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            build-essential \
            bzip2 \
            libgeoip-dev \
            libmaxminddb-dev \
            python3-dev \
            libevent-dev \
            libssl-dev \
            lxml \
            libxml2-dev \
            libxslt1-dev \
            libjpeg-dev \
            libfreetype6-dev \
            liblcms2-dev \
            libtiff5-dev \
            tk-dev \
            tcl-dev \
            psutil \
            linux-headers-amd64 \
            libldap2-dev \
            libsasl2-dev \
            ca-certificates \
            curl \
            dirmngr \
            fonts-noto-cjk \
            gnupg \
            libssl1.0-dev \
            node-less \
            python3-pip \
            python3-pyldap \
            python3-qrcode \
            python3-renderpm \
            python3-setuptools \
            python3-vobject \
            python3-watchdog \
            python-websocket \
            xz-utils \
        && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb \
        && echo '7e35a63f9db14f93ec7feeb0fce76b30c08f2057 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb\
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Grab run deps
RUN set -x; \
    apt-get -qq update && apt-get -qq install -y --no-install-recommends \
    apt-transport-https \
    gnupg2 \
    locales \
    fontconfig \
    libjpeg62-turbo \
    liblcms2-2 \
    libldap-2.4-2 \
    libsasl2-2 \
    libx11-6 \
    libxext6 \
    libxrender1 \
    libxslt1.1 \
    zlib1g \
    zlibc

# install latest postgresql-client
RUN set -x; \
        echo 'deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main' > etc/apt/sources.list.d/pgdg.list \
        && export GNUPGHOME="$(mktemp -d)" \
        && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
        && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
        && gpg --armor --export "${repokey}" | apt-key add - \
        && gpgconf --kill all \
        && rm -rf "$GNUPGHOME" \
        && apt-get update  \
        && apt-get install -y postgresql-client \
        && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian stretch)
RUN set -x;\
    echo "deb http://deb.nodesource.com/node_8.x stretch main" > /etc/apt/sources.list.d/nodesource.list \
    && export GNUPGHOME="$(mktemp -d)" \
    && repokey='9FD3B784BC1C6FC31A8A0A1C1655A0AB68576280' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --armor --export "${repokey}" | apt-key add - \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g rtlcss \
    && rm -rf /var/lib/apt/lists/*

# Install Odoo
ENV ODOO_VERSION 12.0
ENV ODOO_BASEPATH /etc/odoo
RUN git clone --depth=1 -b ${ODOO_VERSION} https://github.com/odoo/odoo.git ${ODOO_BASEPATH}
RUN pip install -r ${ODOO_BASEPATH}/requirements.txt

# Copy entrypoint script and Odoo configuration file
RUN pip3 install num2words xlwt
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc
RUN chown odoo /etc/odoo.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
