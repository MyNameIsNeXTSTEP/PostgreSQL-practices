FROM postgres:16
RUN apt-get update && apt-get -y install postgresql-16-cron && \ 
    apt-get clean && rm -rf /var/lib/apt/lists/*
RUN echo "shared_preload_libraries='pg_cron'" >> /usr/share/postgresql/postgresql.conf.sample && \
    echo "cron.database_name='postgres'" >> /usr/share/postgresql/postgresql.conf.sample