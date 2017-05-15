YADC_CLONE for ARM architecture (Raspberry Pi et al.)
===

Cloned from [CouchDB's official Docker image](https://hub.docker.com/_/couchdb/) ([src](https://github.com/apache/couchdb-docker)), minimal changes applied to `Dockerfile` (e.g. using [resin/rpi-raspbian:jessie](https://hub.docker.com/r/resin/rpi-raspbian/), ...)

Yet Another Dockerized CouchDB – **but for ARM architecture**.
Put the couch in a docker container and ship it anywhere **w/ your ARM architecture device, e.g. Raspberry Pi**.

Build and tested on *Raspberry Pi 3 Model B* (`ARMv8`).

---

- Version (stable): `CouchDB 1.6.1`, `Erlang 17.3`
- Version (stable): `CouchDB 2.0.0`, `Erlang 17.3`

## Available tags

- `1.6.1`: CouchDB 1.6.1
- `1.6.1-couchperuser`: CouchDB 1.6.1 with couchperuser plugin
- `latest`, `2.0.0`: CouchDB 2.0 single node

## Features

* built on top of the solid and small `resin/rpi-raspbian:jessie` base image
* exposes CouchDB on port `5984` of the container
* runs everything as user `couchdb` (security ftw!)
* docker volume for data

## Run (2.0.0/latest)

Available on the docker registry as [matthiasg/rpi-couchdb:latest](https://index.docker.io/u/matthiasg/rpi-couchdb/).
This is a build of the CouchDB 2.0 release. A data volume
is exposed on `/opt/couchdb/data`, and the node's port is exposed on `5984`.

Please note that CouchDB no longer autocreates system tables for you, so you will
have to create `_global_changes`, `_metadata`, `_replicator` and `_users` manually (the admin interface has a "Setup" menu that does this for you).
The node will also start in [admin party mode](http://guide.couchdb.org/draft/security.html#party)!

```bash
# expose it to the world on port 5984 and use your current directory as the CouchDB Database directory
[sudo] docker run -p 5984:5984 -v $(pwd):/opt/couchdb/data matthiasg/rpi-couchdb
18:54:48.780 [info] Application lager started on node nonode@nohost
18:54:48.780 [info] Application couch_log_lager started on node nonode@nohost
18:54:48.780 [info] Application couch_mrview started on node nonode@nohost
18:54:48.780 [info] Application couch_plugins started on node nonode@nohost
[...]
```

Note that you can also use the NODENAME environment variable to set the name of the CouchDB node inside the container.
Once running, you can visit the new admin interface at `http://dockerhost:5984/_utils/`

## Run (1.6.1)

Available on the docker registry as [matthiasg/rpi-couchdb:1.6.1](https://index.docker.io/u/matthiasg/rpi-couchdb/).

```bash
[sudo] docker pull matthiasg/rpi-couchdb:1.6.1

# expose it to the world on port 5984
[sudo] docker run -d -p 5984:5984 --name couchdb matthiasg/rpi-couchdb:1.6.1

curl http://localhost:5984
```

...or with mounted volume for the data

```bash
# expose it to the world on port 5984 and use your current directory as the CouchDB Database directory
[sudo] docker run -d -p 5984:5984 -v $(pwd):/usr/local/var/lib/couchdb --name couchdb matthiasg/rpi-couchdb:1.6.1
```

If you want to provide your own config, you can either mount a directory at `/usr/local/etc/couchdb`
or extend the image and `COPY` your `config.ini` (see [Build you own](#build-your-own)).

If you need (or want) to run couchdb in `net=host` mode, you can customize the port and bind address using environment variables:

 - `COUCHDB_HTTP_BIND_ADDRESS` (default: `0.0.0.0`)
 - `COUCHDB_HTTP_PORT` (default: `5984`)

### with couchperuser plugin

This build includes the `couchperuser` plugin.
`couchperuser` is a CouchDB plugin daemon that creates per-user databases [github.com/etrepum/couchperuser](https://github.com/etrepum/couchperuser).

```
[sudo] docker run -d -p 5984:5984 --name couchdb matthiasg/rpi-couchdb:1.6.1-couchperuser
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

For the `2.0-single` image, configuration is stored at `/opt/couchdb/etc/`.

## Feedback, Issues, Contributing

**Please use Github issues for any questions, bugs, feature requests. :)**
I don't get notified about comments on Docker Hub, so I might respond really late...or not at all.

## Credits, Attribution, THANK YOU and ❤

- [All the fine people contributing to the official image](https://github.com/apache/couchdb-docker/graphs/contributors)
