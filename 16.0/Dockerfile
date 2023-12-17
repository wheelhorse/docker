FROM debian:bullseye
MAINTAINER  Fengbo Wu <fengbo_wu@163.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

RUN echo "deb http://mirrors.tencent.com/debian/ bullseye main non-free contrib"                 > /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tencent.com/debian/ bullseye main non-free contrib"             >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tencent.com/debian-security/ bullseye-security main"                >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tencent.com/debian-security/ bullseye-security main"            >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tencent.com/debian/ bullseye-updates main non-free contrib"         >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tencent.com/debian/ bullseye-updates main non-free contrib"     >> /etc/apt/sources.list
RUN echo "deb http://mirrors.tencent.com/debian/ bullseye-backports main non-free contrib"       >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.tencent.com/debian/ bullseye-backports main non-free contrib"   >> /etc/apt/sources.list

COPY ./wkhtmltox.deb .
# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends --fix-missing\
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
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
        python3-paramiko \
        xz-utils \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ bullseye-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
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

# Install Odoo
ENV ODOO_VERSION 16.0
ARG ODOO_RELEASE=20230430
ARG ODOO_SHA=af1128e5d8126e079a968ec22696a122965a4404
COPY ./odoo.deb .

RUN apt-get update \
    && apt-get -y install --no-install-recommends ./odoo.deb \
    && rm -rf /var/lib/apt/lists/* odoo.deb

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
