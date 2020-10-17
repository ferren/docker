FROM ubuntu:20.04
MAINTAINER Ferren Liu <ferrenliu@163.com>
RUN  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
COPY wkhtmltox_0.12.5-1.bionic_amd64.deb /wkhtmltox.deb
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
	build-essential \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
	libjpeg-dev \
	libldap2-dev \
	libsasl2-dev \
        libssl-dev \
	libxml2-dev \
	libxslt-dev \
	logrotate \
        node-less \
        npm \
	openjdk-11-jdk \
	python3-dev \
        python3-num2words \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        xz-utils \
	zlib1g-dev \
    && apt-get install -y --no-install-recommends /wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* /wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client libpq-dev \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

COPY requirements.txt /requirements.txt
RUN  pip3 install --upgrade pip \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
RUN  pip3 install -r requirements.txt \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com && rm /requirements.txt

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/
COPY ./logrotate.txt /opt/

# set odoo user
ENV ODOO_CONFIGURATION_FILE=/etc/odoo/odoo.conf
ENV ODOO_DATA_DIR=/var/lib/odoo
ENV ODOO_BASE_DIR=/opt/odoo
ENV ODOO_LOG_DIR=/var/log/odoo
ENV ODOO_USER="odoo"
ENV ODOO_GROUP="odoo"
RUN addgroup --gid 1006 $ODOO_GROUP \
    && adduser --system --home $ODOO_DATA_DIR --uid 1006 --gid 1006 --quiet $ODOO_USER \
    && usermod -a -G crontab $ODOO_USER \
    && chown $ODOO_USER:$ODOO_GROUP /opt/logrotate.txt \
    && echo $ODOO_USER > /etc/cron.allow \
    && chown $ODOO_USER:$ODOO_GROUP /var/lib/logrotate \
    && chown $ODOO_USER $ODOO_CONFIGURATION_FILE \
    && chmod 0640 $ODOO_CONFIGURATION_FILE \
    && mkdir -p $ODOO_LOG_DIR \
    && chown $ODOO_USER $ODOO_LOG_DIR \
    && chmod 0750 $ODOO_LOG_DIR \
    && chown $ODOO_USER $ODOO_DATA_DIR \
    && mkdir -p $ODOO_BASE_DIR \
    && chown $ODOO_USER $ODOO_BASE_DIR \
    && touch $ODOO_BASE_DIR/odoo-bin \
    && ln -sf $ODOO_BASE_DIR/odoo-bin /usr/bin/odoo

RUN mkdir -p /etc/logrotate.d \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/etc/odoo", "/var/lib/odoo", "/var/log/odoo/", "/opt/odoo", "/mnt/extra-addons"]

COPY ./logrotate.d/odoo /etc/logrotate.d/odoo

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo", "--logfile", "/var/log/odoo/odoo-server.log"]
