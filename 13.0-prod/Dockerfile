FROM ubuntu:18.04
MAINTAINER Ferren Liu "ferrenliu@maxtrys.com"
RUN  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV LANG C.UTF-8
RUN  sed -i '2a\deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '3a\deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '4a\deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '5a\deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '6a\deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '7a\deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '8a\deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '9a\deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '10a\deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse' /etc/apt/sources.list
RUN  sed -i '11a\deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse' /etc/apt/sources.list
RUN  apt-get update
RUN  apt-get -y upgrade
RUN  apt-get install -y --no-install-recommends ca-certificates \
    build-essential \
    curl \
    dirmngr \
    fonts-noto-cjk \
    gnupg \
    libssl1.0-dev \
    xz-utils \
    postgresql-client \
    libsasl2-dev \
    libldap2-dev \
    libxslt-dev \
    libxml2-dev \
    nodejs-dev \
    node-gyp \
    npm \
    wkhtmltopdf \
    libxrender1 \
    xfonts-base \
    xfonts-75dpi \
    python3-dev \
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
    python-xlwt \
    python-ldap \
    python-lxml 
COPY wkhtmltox_0.12.5-1.bionic_amd64.deb /wkhtmltox.deb
COPY requirements.txt /requirements.txt
COPY ./odoo.conf /etc/odoo/

RUN  mkdir -p /opt/odoo/ && mkdir /var/lib/odoo
VOLUME ["/opt/odoo", "/etc/odoo",  "/var/lib/odoo"]

RUN  dpkg -i /wkhtmltox.deb && rm /wkhtmltox.deb
RUN set -x; \
    npm install -g rtlcss
RUN  pip3 install --upgrade pip \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
RUN  pip3 install -r requirements.txt \
    -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com && rm /requirements.txt

EXPOSE 8069 8071

ENV ODOO_RC /etc/odoo/odoo.conf

ENTRYPOINT [ "python3" ]
CMD [ "/opt/odoo/odoo-bin" ]
