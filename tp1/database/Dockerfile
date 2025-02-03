FROM postgres:14.1-alpine

#ENV POSTGRES_DB=db \
# POSTGRES_USER=usr \
# POSTGRES_PASSWORD=pwd

COPY SQL-init/CreateScheme.sql /docker-entrypoint-initdb.d/CreateScheme.sql
COPY SQL-init/InsertData.sql /docker-entrypoint-initdb.d/InsertData.sql