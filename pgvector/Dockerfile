FROM postgres:18-alpine

RUN apk add --no-cache --virtual .build-deps \
    git \
    build-base \
    clang15 \
    llvm15-dev \
    && git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git /tmp/pgvector \
    && cd /tmp/pgvector \
    && make \
    && make install \
    && cd / \
    && rm -rf /tmp/pgvector \
    && apk del .build-deps

COPY init.sql /docker-entrypoint-initdb.d/

ENV POSTGRES_DB=mydb
ENV POSTGRES_USER=myuser
ENV POSTGRES_PASSWORD=mypassword