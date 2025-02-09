FROM postgres:14.1-alpine

# Variables d'environnement
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
# COPY sql file dans le conteur postgres
COPY data/CreateScheme.sql /docker-entrypoint-initdb.d/
COPY data/InsertData.sql /docker-entrypoint-initdb.d/