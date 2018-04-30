YADC_CLONE for ARM architecture (Raspberry Pi et al.)
===

Cloned from [CouchDB's official Docker image](https://hub.docker.com/_/couchdb/) ([src (`upstream`)](https://github.com/apache/couchdb-docker)), minimal changes applied to `Dockerfile` (e.g. using [resin/rpi-raspbian:jessie](https://hub.docker.com/r/resin/rpi-raspbian/), ...)

Yet Another Dockerized CouchDB – **but for ARM architecture**.
Put the couch in a docker container and ship it anywhere **w/ your ARM architecture device, e.g. Raspberry Pi**.

Build and tested on *Raspberry Pi 3 Model B* (`ARMv8`).

---

- Version (stable): `CouchDB 1.7.1`, `Erlang 17.3`
- Version (stable): `CouchDB 2.1.1`, `Erlang 17.3`

## Available tags

- `1.7.1`: CouchDB 1.7.1
- `1.7.1-couchperuser`: CouchDB 1.7.1 with couchperuser plugin
- `latest`, `2.1.1`: CouchDB 2.1.1 single node (capable of running in a cluster)

## Features

* built on top of the solid and small `resin/rpi-raspbian:jessie` base image
* exposes CouchDB on port `5984` of the container
* runs everything as user `couchdb` (security ftw!)
* docker volume for data

## Run (latest/2.1.1)

Available on the docker registry as [matthiasg/rpi-couchdb:latest](https://hub.docker.com/r/matthiasg/rpi-couchdb/).
This is a build of the CouchDB 2.0 release. A data volume
is exposed on `/opt/couchdb/data`, and the node's port is exposed on `5984`.

By default, CouchDB's HTTP interface is exposed on port `5984`. Once running, you can visit the new admin interface at `http://<dockerhost>:5984/_utils/`

CouchDB uses `/opt/couchdb/data` to store its data, and is exposed as a volume.

Here is an example launch line for a single-node CouchDB with an admin username and password of `admin` and `password`, exposed to the world on port `5984`:

```bash
# expose it to the world on port 5984 and use your current directory as the CouchDB Database directory
$ docker run -p 5984:5984 --volume $(pwd):/opt/couchdb/data --volume ~/etc/local.d:/opt/couchdb/etc/local.d --env COUCHDB_USER=admin --env COUCHDB_PASSWORD=password matthiasg/rpi-couchdb:2.1.1
```

### Detailed configuration (latest/2.x)

CouchDB uses `/opt/couchdb/etc/local.d` to store its configuration. It is highly recommended to bind map this to an external directory, to persist the configuration across restarts.

CouchDB also uses `/opt/couchdb/etc/vm.args` to store Erlang runtime-specific changes. Changing these values is less common. If you need to change the epmd port, for instance, you will want to bind mount this file as well. (Note: files cannot be bind-mounted on Windows hosts.)
Available on the docker registry as [matthiasg/rpi-couchdb:1.7.1](https://hub.docker.com/r/matthiasg/rpi-couchdb/).

In addition, a few environment variables are provided to set very common parameters:

* `COUCHDB_USER` and `COUCHDB_PASSWORD` will create an ini-file based local admin user with the given username and password in the file `/opt/couchdb/etc/local.d/docker.ini`.
* `COUCHDB_SECRET` will set the CouchDB shared cluster secret value, in the file `/opt/couchdb/etc/local.d/docker.ini`.
* `NODENAME` will set the name of the CouchDB node inside the container to `couchdb@${NODENAME}`, in the file `/opt/couchdb/etc/vm.args`. This is used for clustering purposes and can be ignored for single-node setups.

If other configuration settings are desired, externally mount `/opt/couchdb/etc` and provide `.ini` configuration files under the `/opt/couchdb/etc/local.d` directory.

### Important notes (latest/2.x)

Please note that CouchDB no longer autocreates system databases for you. This is intentional; multi-node CouchDB deployments must be joined into a cluster before creating these databases.

You must create `_global_changes`, `_metadata`, `_replicator` and `_users` after the cluster has been fully configured. (The Fauxton UI has a "Setup" wizard that does this for you.)

The node will also start in [admin party mode](http://guide.couchdb.org/draft/security.html#party)!

Note also that port 5986 is not exposed, as this can present *significant* security risks. We recommend either connecting to the node directly to access this port, via `docker exec -it <instance> /bin/bash` and accessing port 5986, or use of `--expose 5986` when launching the container, but **ONLY** if you do not expose this port publicly. Port 5986 is scheduled to be removed with the 3.x release series.

## Run (1.7.1)

Available as an image on Docker Hub as [matthiasg/rpi-couchdb:1.7.1](https://hub.docker.com/r/matthiasg/rpi-couchdb/)

```bash
[sudo] docker pull matthiasg/rpi-couchdb:1.7.1

# expose it to the world on port 5984
[sudo] docker run -d -p 5984:5984 --name couchdb matthiasg/rpi-couchdb:1.7.1

curl http://localhost:5984
```

...or with mounted volume for the data

```bash
# expose it to the world on port 5984 and use your current directory as the CouchDB Database directory
[sudo] docker run -d -p 5984:5984 -v $(pwd):/usr/local/var/lib/couchdb --name couchdb matthiasg/rpi-couchdb:1.7.1
```

If you want to provide your own config, you can either mount a directory at `/usr/local/etc/couchdb`
or extend the image and `COPY` your `config.ini` (see [Build you own](#build-your-own)).

If you need (or want) to run couchdb in `net=host` mode, you can customize the port and bind address using environment variables:

 - `COUCHDB_HTTP_BIND_ADDRESS` (default: `0.0.0.0`)
 - `COUCHDB_HTTP_PORT` (default: `5984`)

### 1.7.1 with couchperuser plugin

This build includes the `couchperuser` plugin.
`couchperuser` is a CouchDB plugin daemon that creates per-user databases [github.com/etrepum/couchperuser](https://github.com/etrepum/couchperuser).

```
[sudo] docker run -d -p 5984:5984 --name couchdb matthiasg/rpi-couchdb:1.7.1-couchperuser
```

## Build your own

You can use `matthiasg/rpi-couchdb` as the base image for your own couchdb instance.
You might want to provide your own version of the following files:

* `local.ini` for your custom CouchDB config

Example Dockerfile:

```
FROM matthiasg/rpi-couchdb:latest

COPY local.ini /usr/local/etc/couchdb/local.d/
```

and then build and run

```
[sudo] docker build -t you/awesome-couchdb .
[sudo] docker run -d -p 5984:5984 -v ~/couchdb:/usr/local/var/lib/couchdb you/awesome-couchdb
```

For the `2` image, configuration is stored at `/opt/couchdb/etc/`.

## Add Upstream, fetching changes

```sh
git remote add upstream https://github.com/apache/couchdb-docker.git
git fetch origin -v; git fetch upstream -v; git merge upstream/master
```

## Feedback, Issues, Contributing

General feedback is welcome at our [user][1] or [developer][2] mailing lists.

## Credits, Attribution, THANK YOU and ❤

- [All the fine people contributing to the official image](https://github.com/apache/couchdb-docker/graphs/contributors)

[1]: http://mail-archives.apache.org/mod_mbox/couchdb-user/
[2]: http://mail-archives.apache.org/mod_mbox/couchdb-dev/
[3]: https://github.com/apache/couchdb/blob/master/CONTRIBUTING.md
[4]: http://www.apache.org/dev/release-distribution.html#unreleased
