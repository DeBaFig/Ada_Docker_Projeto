FROM postgres:14.3

WORKDIR /docker-entrypoint-initdb.d

ADD schema.sql .

VOLUME [ "db-data:/var/lib/postgresql/data" ]

EXPOSE 5432

## psql -h localhost -p 5432  -U postgres -d DB_ADA
## docker exec -it postgres_ada pg_dump -c -U postgres -d DB_ADA > dump_`date +%Y-%m-%d"_"%H_%M_%S`.sql