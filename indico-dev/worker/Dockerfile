FROM python:3.10-bullseye

ENV INDICO_VIRTUALENV="/opt/indico/.venv" INDICO_CONFIG="/opt/indico/etc/indico.conf"

ARG pip="${INDICO_VIRTUALENV}/bin/pip"

USER root

# Install dependencies
RUN set -ex && \
    apt-get update
    # apt-get upgrade

# Instalación de nodejs
# RUN curl -sL https://deb.nodesource.com/setup_16.x | bash -
# RUN apt remove nodejs && \
    # rm -rf /usr/local/bin/node* && \
    # rm -rf /usr/local/bin/npm* && \
    # rm -rf /etc/apt/sources.list.d/nodesource.list && \
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash -

RUN apt-get install -y nodejs


# curl git wget libpq-dev | dice que están en su última versión
RUN apt install -y postgresql-client vim less bash  gettext zip  && \
    apt install -y --install-recommends libxslt1-dev libxml2-dev libffi-dev libpcre3-dev libyaml-dev build-essential redis-server postgresql libpq-dev && \
    apt install -y libjpeg62-turbo-dev && \
    apt install -y gcc g++ make musl-dev
    #  mupdf-dev freetype-dev
    # apt-get -y install linux-headers-$(uname -r)
    # gcc, g++, make, musl-dev, linux-headers, mupdf-dev and freetype-dev are needed to build some python and/or node libraries


# Install maildump
#RUN pip install --no-cache-dir pipx && \
#    pipx install maildump && \
#    pip uninstall pipx -y

# Create a virtual environment
RUN python -m venv ${INDICO_VIRTUALENV}
RUN ${pip} install --no-cache-dir --upgrade pip uwsgi

# Install the Transifex client
RUN mkdir -p /opt/tx
RUN cd /opt/tx && curl -o- https://raw.githubusercontent.com/transifex/cli/master/install.sh | bash

WORKDIR /opt/indico

# Clone indico
RUN git clone https://github.com/indico/indico.git src --depth 1

# Clone indico-plugins
RUN git clone https://github.com/indico/indico-plugins.git indico-plugins --depth 1

# Install requirements
RUN cd /opt/indico/src && ${pip} install --no-cache-dir -e '.[dev]'
RUN for dir in /opt/indico/indico-plugins/*/; do (cd "${dir}" && ${pip} install --no-cache-dir -e '.[dev]'); done

# Run webpack for indico and plugins
RUN cd /opt/indico/src && npm ci && \
    ${INDICO_VIRTUALENV}/bin/python ./bin/maintenance/build-assets.py indico --dev && \
    ${INDICO_VIRTUALENV}/bin/python ./bin/maintenance/build-assets.py all-plugins --dev /opt/indico/indico-plugins && \
    npm cache clean --force

RUN ["/bin/bash", "-c", "mkdir -p --mode=775 /opt/indico/{etc,tmp,log,cache,archive}"]

RUN ${INDICO_VIRTUALENV}/bin/indico setup create-symlinks /opt/indico
RUN ${INDICO_VIRTUALENV}/bin/indico setup create-logging-config /opt/indico/etc

COPY .transifexrc /opt/indico/etc/

WORKDIR /opt/indico

# uwsgi & maildump ports
#EXPOSE 59999 60000
EXPOSE 59999


COPY uwsgi.ini /etc/uwsgi.ini

COPY run_indico.sh run_celery.sh pull_translations.sh run_initial_setup.py /opt/indico/
RUN chmod 755 /opt/indico/*.sh
