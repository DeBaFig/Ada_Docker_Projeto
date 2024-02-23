FROM postgres:14.3

WORKDIR /docker-entrypoint-initdb.d

COPY  schema.sql /docker-entrypoint-initdb.d

VOLUME [ "db-data:/var/lib/postgresql/data" ]

EXPOSE 5432
