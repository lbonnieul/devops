FROM postgres:14.1-alpine

COPY SQL-init/CreateScheme.sql /docker-entrypoint-initdb.d/CreateScheme.sql
COPY SQL-init/InsertData.sql /docker-entrypoint-initdb.d/InsertData.sql
