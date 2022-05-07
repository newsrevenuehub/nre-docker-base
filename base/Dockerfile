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
