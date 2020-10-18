FROM debian:buster-slim
MAINTAINER Ferren Liu <ferrenliu@163.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

RUN sed -i 's/deb.debian.org/ftp2.cn.debian.org/g' /etc/apt/sources.list

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
COPY wkhtmltox_0.12.5-1.stretch_amd64.deb /wkhtmltox.deb
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
	build-essential \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
	git \
        libssl-dev \
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
    && apt-get install -y --no-install-recommends /wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* /wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ buster-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

COPY requirements.txt /requirements.txt
RUN  pip3 install --upgrade pip \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
RUN  pip3 install -r requirements.txt \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com && rm /requirements.txt

# Copy Odoo configuration file
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /var/lib/odoo \
    && mkdir -p /opt/odoo \
    && mkdir -p /mnt/extra-addons
VOLUME ["/etc/odoo", "/var/lib/odoo", "/opt/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

ENTRYPOINT ["/bin/bash"]
