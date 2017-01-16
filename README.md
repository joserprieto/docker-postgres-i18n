# Supported tags and respective `Dockerfile` links

-   [`9.6.1-es-es`, `9.6-es-es`, `9-es-es`, `latest-es-es`, `latest` (*9.6/Debian/Dockerfile*)](https://github.com/joserprieto/docker-postgres-i18n/blob/master/9.6/debian/Dockerfile)
-   [`9.4.10-es-es`, `9.4-es-es` (*9.4/Debian/Dockerfile*)](https://github.com/joserprieto/docker-postgres-i18n/blob/master/9.4/debian/Dockerfile)
-   [`9.2.19-es-es`, `9.2-es-es` (*9.2/Debian/Dockerfile*)](https://github.com/joserprieto/docker-postgres-i18n/blob/master/9.2/debian/Dockerfile)

For more information about this image and its history, please see [the relevant manifest file (`library/postgres`)](https://github.com/docker-library/official-images/blob/master/library/postgres). This image is updated via [pull requests to the `docker-library/official-images` GitHub repo](https://github.com/docker-library/official-images/pulls?q=label%3Alibrary%2Fpostgres).

For detailed information about the virtual/transfer sizes and individual layers of each of the above supported tags, please see [the `repos/postgres/tag-details.md` file](https://github.com/docker-library/repo-info/blob/master/repos/postgres/tag-details.md) in [the `docker-library/repo-info` GitHub repo](https://github.com/docker-library/repo-info).

# What is PostgreSQL?

PostgreSQL, often simply "Postgres", is an object-relational database management system (ORDBMS) with an emphasis on extensibility and standards-compliance. As a database server, its primary function is to store data, securely and supporting best practices, and retrieve it later, as requested by other software applications, be it those on the same computer or those running on another computer across a network (including the Internet). It can handle workloads ranging from small single-machine applications to large Internet-facing applications with many concurrent users. Recent versions also provide replication of the database itself for security and scalability.

PostgreSQL implements the majority of the SQL:2011 standard, is ACID-compliant and transactional (including most DDL statements) avoiding locking issues using multiversion concurrency control (MVCC), provides immunity to dirty reads and full serializability; handles complex SQL queries using many indexing methods that are not available in other databases; has updateable views and materialized views, triggers, foreign keys; supports functions and stored procedures, and other expandability, and has a large number of extensions written by third parties. In addition to the possibility of working with the major proprietary and open source databases, PostgreSQL supports migration from them, by its extensive standard SQL support and available migration tools. And if proprietary extensions had been used, by its extensibility that can emulate many through some built-in and third-party open source compatibility extensions, such as for Oracle.

> [wikipedia.org/wiki/PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL)

![logo](./logo.png)

# About this image

This image is based on PostgreSQL Official Image:

> [PostgreSQL Official Image](https://hub.docker.com/_/postgres/)

> [PostgreSQL Official Image Repository](https://github.com/docker-library/postgres/)

But add extra support for configure timezone and locales before built the image, with the arguments in 
the `docker build` command (`--build-arg`), and `ARG` in the `Dockerfile`.

Default values for the timezone and locale are:

```bash
Timezone:   "Europe/Madrid"
Locale:     es_ES.UTF-8
```

# Locale and timezone.

## Build a localized image.

To build a localized image, this is, a PostgreSQL (Debian based) Image with locale and timezone correctly defined, we
have to use the `--build-arg` parameter of the `docker build` command.

An example:

```bash
docker build --build-arg TIMEZONE="Europe/France" --build-arg LOCALE_LANG_COUNTRY="fr_FR" .
```

## Locales

The locales are generated with the snippet described in Debian Official Image, and
[used in PostgreSQL Official Image](https://github.com/docker-library/postgres/blob/69bc540ecfffecce72d49fa7e4a46680350037f9/9.6/Dockerfile#L21-L24):

```dockerfile
RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8
```

We only add a few arguments for the build:

```dockerfile
ARG LOCALE_LANG_COUNTRY="es_ES"
ARG LOCALE_CODIFICATION="UTF-8"
ARG LOCALE_CODIFICATION_ENV="utf8"
```

And execute the build as:


```dockerfile
    && echo "=> Configuring and installing locale (${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION}):" \
    && apt-get install -y locales \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i ${LOCALE_LANG_COUNTRY} -c -f ${LOCALE_CODIFICATION} -A /usr/share/locale/locale.alias ${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION} \
```

## Timezone

We use a snippet provided by [Oscar](https://oscarmlage.com/) (Thanks!):

```dockerfile
    echo "=> Configuring and installing timezone:" && \
        echo "Europe/Madrid" > /etc/timezone && \
        dpkg-reconfigure -f noninteractive tzdata && \
```

Only added an ARG for the build:

```dockerfile
ARG TIMEZONE="Europe/Madrid"
```

And, finally:

```dockerfile
    && echo "=> Configuring and installing timezone (${TIMEZONE}):" \
    && echo ${TIMEZONE} > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata \
```

Obviously, the value of the timezone has to be one of the right values:

[Change Timezone on Debian](https://wiki.debian.org/TimeZoneChanges)
[List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

## Why rewrite all the Dockerfile?

The Dockerfile's of each version, are the same as de Official Image, and only adds the code explained before; so, why
not do a Dockerfile like:
```dockerfile
FROM postgres:latest
..
```
And the code for the localization?

Because, as the official documentation of PostgreSQL explains:

[https://www.postgresql.org/docs/9.4/static/charset.html]()

[https://www.postgresql.org/docs/9.4/static/locale.html#AEN35162]()

So, the `initdb` commmand is executed after the apt-get install; and PostgreSQL was installed with the deb packages
system of Debian; so, we have to localized the image before the installation of the PostgreSQL.

# How to use this image.

## start a postgres instance

```console
$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

This image includes `EXPOSE 5432` (the postgres port), so standard container linking will make it automatically available to the linked containers. The default `postgres` user and database are created in the entrypoint with `initdb`.

> The postgres database is a default database meant for use by users, utilities and third party applications.
> [postgresql.org/docs](http://www.postgresql.org/docs/9.5/interactive/app-initdb.html)

## connect to it from an application

```console
$ docker run --name some-app --link some-postgres:postgres -d application-that-uses-postgres
```

## ... or via `psql`

```console
$ docker run -it --rm --link some-postgres:postgres postgres psql -h postgres -U postgres
psql (9.5.0)
Type "help" for help.

postgres=# SELECT 1;
 ?column?
----------
        1
(1 row)

```

## Environment Variables

The PostgreSQL image uses several environment variables which are easy to miss. While none of the variables are required, they may significantly aid you in using the image.

### `POSTGRES_PASSWORD`

This environment variable is recommended for you to use the PostgreSQL image. This environment variable sets the superuser password for PostgreSQL. The default superuser is defined by the `POSTGRES_USER` environment variable. In the above example, it is being set to "mysecretpassword".

### `POSTGRES_USER`

This optional environment variable is used in conjunction with `POSTGRES_PASSWORD` to set a user and its password. This variable will create the specified user with superuser power and a database with the same name. If it is not specified, then the default user of `postgres` will be used.

### `PGDATA`

This optional environment variable can be used to define another location - like a subdirectory - for the database files. The default is `/var/lib/postgresql/data`, but if the data volume you're using is a fs mountpoint (like with GCE persistent disks), Postgres `initdb` recommends a subdirectory (for example `/var/lib/postgresql/data/pgdata` ) be created to contain the data.

### `POSTGRES_DB`

This optional environment variable can be used to define a different name for the default database that is created when the image is first started. If it is not specified, then the value of `POSTGRES_USER` will be used.

### `POSTGRES_INITDB_ARGS`

This optional environment variable can be used to send arguments to `postgres initdb`. The value is a space separated string of arguments as `postgres initdb` would expect them. This is useful for adding functionality like data page checksums: `-e POSTGRES_INITDB_ARGS="--data-checksums"`.

# How to extend this image

If you would like to do additional initialization in an image derived from this one, add one or more `*.sql` or `*.sh` scripts under `/docker-entrypoint-initdb.d` (creating the directory if necessary). After the entrypoint calls `initdb` to create the default `postgres` user and database, it will run any `*.sql` files and source any `*.sh` scripts found in that directory to do further initialization before starting the service.

For example, to add an additional user and database, add the following to `/docker-entrypoint-initdb.d/init-user-db.sh`:

```bash
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" <<-EOSQL
	CREATE USER docker;
	CREATE DATABASE docker;
	GRANT ALL PRIVILEGES ON DATABASE docker TO docker;
EOSQL
```

These initialization files will be executed in sorted name order as defined by the current locale, which defaults to `en_US.utf8`. Any `*.sql` files will be executed by `POSTGRES_USER`, which defaults to the `postgres` superuser. It is recommended that any `psql` commands that are run inside of a `*.sh` script be executed as `POSTGRES_USER` by using the `--username "$POSTGRES_USER"` flag. This user will be able to connect without a password due to the presence of `trust` authentication for Unix socket connections made inside the container.

You can also extend the image with a simple `Dockerfile` to set a different locale. The following example will set the default locale to `de_DE.utf8`:

```dockerfile
FROM postgres:9.4
RUN localedef -i de_DE -c -f UTF-8 -A /usr/share/locale/locale.alias de_DE.UTF-8
ENV LANG de_DE.utf8
```

Since database initialization only happens on container startup, this allows us to set the language before it is created.

# Caveats

If there is no database when `postgres` starts in a container, then `postgres` will create the default database for you. While this is the expected behavior of `postgres`, this means that it will not accept incoming connections during that time. This may cause issues when using automation tools, such as `fig`, that start several containers simultaneously.

## Issues

If you have any problems with or questions about this image, or through a [GitHub issue](https://github.com/joserprieto/docker-postgres-i18n
/issues).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/joserprieto/docker-postgres-i18n
/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.

## Documentation

To define how and where will be stored....