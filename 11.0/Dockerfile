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
            libxml2-dev \
            libxslt1-dev \
            libjpeg-dev \
            libfreetype6-dev \
            liblcms2-dev \
            libtiff5-dev \
            tk-dev \
            tcl-dev \
            linux-headers-amd64 \
            libldap2-dev \
            libsasl2-dev \
            ca-certificates \
            curl \
            dirmngr \
            fonts-noto-cjk \
            gnupg \
            node-less \
            python3-pip \
            python3-pyldap \
            python3-qrcode \
            python3-renderpm \
            python3-setuptools \
            python3-vobject \
            python3-watchdog \
            xz-utils \
            python-websocket \
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

# Grab latest git            //-- to `pip install` customized python packages & apply patches
RUN apt-get -qq update && apt-get -qq install -y --no-install-recommends git-core

# Grab latest pip
RUN curl --silent --show-error --location https://bootstrap.pypa.io/get-pip.py | python3 /dev/stdin --no-cache-dir

# Install Odoo
ENV ODOO_VERSION 11.0
ENV ODOO_BASEPATH /etc/odoo
RUN git clone --depth=1 -b ${ODOO_VERSION} https://github.com/odoo/odoo.git ${ODOO_BASEPATH}
RUN pip install -r ${ODOO_BASEPATH}/requirements.txt
RUN pip install simplejson zeep>=3.2.0 python-stdnum>=1.11

# Create odoo user and group
ENV ODOO_USER odoo
RUN addgroup --system ${ODOO_USER}
RUN adduser --system --ingroup ${ODOO_USER} --home ${ODOO_BASEPATH} --disabled-login --shell /sbin/nologin ${ODOO_USER}

# Copy entrypoint script and Odoo configuration file
RUN pip3 install num2words xlwt websocket-client openpyxl
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc
RUN chown ${ODOO_USER} /etc/odoo.conf
RUN mkdir /var/lib/odoo && chown -R ${ODOO_USER}:${ODOO_USER} /var/lib/odoo

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R ${ODOO_USER} /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
