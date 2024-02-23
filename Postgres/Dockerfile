FROM postgres:14.3

WORKDIR /docker-entrypoint-initdb.d

ADD ./Postgres/schema.sql /docker-entrypoint-initdb.d

EXPOSE 5432
