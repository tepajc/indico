# Docker containers for Indico

## Production-like setup

[![Build Status](https://www.travis-ci.org/indico/indico-containers.svg?branch=master)](https://www.travis-ci.org/indico/indico-containers)

### Quickstart

To start the containers, run:
```sh
$ cd indico-prod && docker compose up
```

Indico will be accessible at [localhost:8080](localhost:8080). You can also access the wsgi app directly at [localhost:9090](localhost:9090) which skips the nginx proxy.

### Configuration

There are two config files which you can use to configure this setup.
- [prod.env](prod.env) - This file specifies general properties like the DB config, nginx port & server name and the path to `indico.conf`.
- [indico.prod.conf](indico.prod.conf) - This is the standard `indico.conf` file used by Indico itself. It is passed to the containers as a read-only volume.

The production setup contains:
- _indico-web_ - Indico running behind uwsgi ([localhost:9090](localhost:9090))
- _indico-celery_ - Indico celery task runner
- _indico-redis_ - redis
- _indico-nginx_ - nginx proxy ([localhost:8080](localhost:8080))

_indico-web_ uses the `getindico/indico` image from Dockerhub. You can build this image locally using the [build_latest.sh](build_latest.sh) script. The image pulls the latest Indico release from PyPI together with the `indico-plugins` package. You can use your `indico.conf` to specify which plugins you want to enable.

### Different setups

If you don't need the DB and the nginx proxy, you can just run:
```sh
$ docker compose up indico-web
```

This will bring up only Indico, celery and redis. The DB should be on the same network to be accessible.

The `getindico/indico` image can also be used completely standalone outside of this docker-compose setup, as long as the remaining services (postgres, redis, celery) are available elsewhere. In that case, make sure to update `REDIS_CACHE_URL`, `CELERY_BROKER` in your `indico.conf` and the DB connection settings in [prod.env](prod.env).

This is how you can run Indico on its own:
```sh
$ docker run getindico/indico /opt/indico/run_indico.sh
```

Or to run celery from the same image:
```sh
$ docker run getindico/indico /opt/indico/run_celery.sh
```

In the above, we omit the network setup to make e.g. the DB accessible from the containers.

## Development setup

The development setup builds an image from the latest master. Essentially, it puts this [tutorial](https://docs.getindico.io/en/stable/installation/development/) into a Dockerfile. The main purpose of this setup is to allow anyone to relatively quickly and painlessly try out the latest translations from Transifex without having to manually setup an Indico instance.

### Quickstart

To start the containers, run:
```sh
$ cd indico-dev && docker compose up
```
Indico should now be accessible at [localhost:8080](localhost:8080).

The Indico container by default pulls the latest translations from a public mirror which does not need authentication. The mirror updates its translations once every hour. If you want to pull directly from Transifex, you can provide your Transifex API token in the [dev.env](dev.env) file (`TRANSIFEX_API_TOKEN=1/xxx`).

The development setup creates a default admin user with username `admin` and password `indiko42`.

Since emails also get translated, you can use maildump to check those as well. Maildump is available at [localhost:60000](localhost:60000).

Check the [production setup](#production-like-setup) to see more configuration options. 

## OpenShift

```sh
$ oc create configmap settings --from-literal=baseurl=<base_url> --from-literal=pgdatabase=<db_name>
--from-literal=pghost=<db_host> --from-literal=pguser=<db_user> --from-literal=pgport=<db_port>
--from-literal=pgpassword=<db_password> --from-literal=sentrydsn=<sentrydsn> --from-literal=secretkey=<secretkey>
--from-literal=storage=<storage> --from-literal=attachmentstorage=<attachment_storage>


$ cd openshift/
$ ./create.sh
```

In case you want to run the postgres container instead of DBoD (DataBase On Demand), keep in mind to set the `pghost`
literal as `indico-postgres` and the rest of literals accordingly:

```sh
$ oc create configmap settings --from-literal=baseurl=<base_url> --from-literal=pgdatabase=indico
--from-literal=pghost=indico-postgres --from-literal=pguser=indico --from-literal=pgport=5432
--from-literal=pgpassword=indicopass --from-literal=sentrydsn=<sentrydsn> --from-literal=secretkey=<secretkey>
--from-literal=storage=<storage> --from-literal=attachmentstorage=<attachment_storage>
```

If you want to use EOS storage you need to:

- add path to EOS in storage dict (settings configmap), for example:

```
    {'eos' : 'fs:/eos/path/to/folder'}
```

- set attachment storage to one of the defined storages, for example "eos"
