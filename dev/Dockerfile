# syntax=docker/dockerfile:1.4
FROM python:3.9.11

WORKDIR /

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
RUN <<EOF
  RUN_DEPS=libpcre3 mime-support postgresql-client vim curl 
  apt-get update -qq
  apt-get -y -qq install wget gnupg2 lsb-release
  echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list
  wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
  apt-get update -qq
  apt-get -y -qq install ${RUN_DEPS}
  seq 1 8 | xargs -I{} mkdir -p /usr/share/man/man{}
  rm -rf /var/lib/apt/lists/*
EOF

COPY pyproject.toml poetry.lock /

RUN <<EOF
  BUILD_DEPS=build-essential libpcre3-dev libpq-dev 
  apt-get update 
  apt-get install -y --no-install-recommends $BUILD_DEPS
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $BUILD_DEPS
  rm -rf /var/lib/apt/lists/*
EOF
ENV POETRY_VERSION=1.1.13
RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -
ENV PATH "/root/.poetry/bin:$PATH"
RUN <<EOF
  poetry config virtualenvs.create false 
  poetry install --no-dev
EOF
# TODO: why equal on some and blank on others?
ENV NPM_CONFIG_LOGLEVEL warn
ENV NPM_CONFIG_PREFIX=/node_modules
ENV PATH=$PATH:/node_modules/.bin
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE False
# TODO: should this be here or in base? 
ENV PIP_NO_BINARY=psycopg2

# Install ligbnetls30, needed workaround for this issue to install nodesource:  https://github.com/nodesource/distributions/issues/1266
# TODO: see if this is still needed
#RUN apt update && apt install libgnutls30

# Install nodejs from nodesource
# TODO: is there a way to pin this to a particular version?
RUN <<EOF
  DISTRO="$(lsb_release -s -c)"
  echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections 
  echo "deb https://deb.nodesource.com/node_14.x ${DISTRO} main" > /etc/apt/sources.list.d/nodesource.list
  wget -qO- https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - 
  apt-get -qq update
  apt-get -qq install nodejs > /dev/null
  rm -rf /var/lib/apt/lists/*
EOF

WORKDIR /

# we installed everything else in the base image; this installs the dev deps
RUN poetry install

COPY node /node
WORKDIR /node

RUN <<EOF
  npm install
  mv node_modules /
EOF
WORKDIR /