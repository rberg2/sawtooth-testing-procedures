
# Grid Testing Procedure #

## Building code and Docker Images ##

The steps in this section cover building and testing the Grid code and Docker
Images.

### Build local Docker Images ###

First lets build the local Docker Images used to run the compiled code passed
in as a volume.

```
$ docker-compose build --force-rm
```

#### Verification Step ####

```
$ docker images | grep grid
grid_gridd                latest              ffdd23e92432        2 minutes ago       130MB
grid-pike-tp              latest              11f953d9ea56        8 minutes ago       81.8MB
grid-schema-tp            latest              29d9aebd7ad5        14 minutes ago      81.9MB
grid-track-and-trace-tp   latest              609e850b2a9f        18 minutes ago      82.8MB
```

Checklist: Have the following Docker Images been creates?

* grid_gridd
* grid-pike-tp
* grid-schema-tp
* grid-track-and-trace-tp

### Build Debs and Docker Images with Debs installed ###

The steps in this section compile and package the Gid code and builds new
Docker images with this developmental code.

```
$ docker-compose -f docker-compose-installed.yaml build --force-rm
```

#### Verification Step ####

```
$ docker images | grep gridd
gridd-installed                            latest              ead598fb714d        39 seconds ago      154MB
```

Checklist

* Was the docker image gridd-installed created

### Copy the built deb packages out ###

```
$ mkdir -p build/debs
$ docker run --rm -v $(pwd)/build/debs/:/debs gridd-installed:latest bash -c "cp /tmp/grid-daemon*.deb /debs"
```

#### Verification Step ####

```
$ ls -l build/debs 
total 2216
-rw-r--r-- 1 root root 2267572 May 30 13:59 grid-daemon_0.1.0_amd64.deb
```

Checklist

* Does grid-daemon_{VERSION_NUMBER}_amd64.deb exist? 

### docker-compose.yaml ###

```
$ docker-compose -f docker-compose.yaml up
```

#### Verification Step ####

To verify this step has been completed correctly run the following command and
ensure the following Docker Containers are running

```
$ docker ps
CONTAINER ID        IMAGE                                          COMMAND                   CREATED             STATUS              PORTS                              NAMES
8521f3c823d9        hyperledger/sawtooth-settings-tp:1.1           "settings-tp -vv -C …"    31 seconds ago      Up 23 seconds       4004/tcp                           grid-sawtooth-settings-tp
36b34b661e2d        hyperledger/sawtooth-rest-api:1.1              "sawtooth-rest-api -…"    31 seconds ago      Up 26 seconds       4004/tcp, 0.0.0.0:8024->8008/tcp   grid-sawtooth-rest-api
d130283e155d        hyperledger/sawtooth-devmode-engine-rust:1.1   "devmode-engine-rust…"    31 seconds ago      Up 29 seconds                                          sawtooth-devmode-engine-rust-default
49798f946c30        grid-track-and-trace-tp                        "bash -c '\n  /grid-t…"   34 seconds ago      Up 30 seconds                                          grid-track-and-trace-tp
38eb75748981        grid_gridd                                     "bash -c '\n  # we ne…"   34 seconds ago      Up 25 seconds       0.0.0.0:8080->8080/tcp             gridd
db8610fbd7dd        postgres                                       "docker-entrypoint.s…"    34 seconds ago      Up 24 seconds       0.0.0.0:5432->5432/tcp             grid_db_1_3eb10e7984ad
f7b8b2d60c8f        hyperledger/sawtooth-shell:1.1                 "bash -c '\n  sawtoot…"   34 seconds ago      Up 27 seconds       4004/tcp, 8008/tcp                 grid-sawtooth-shell
bbc3ee396083        grid-pike-tp                                   "bash -c '\n  /grid-p…"   34 seconds ago      Up 28 seconds                                          grid-pike-tp
15ba0849cd39        grid-schema-tp                                 "bash -c '\n  /grid-s…"   34 seconds ago      Up 29 seconds                                          grid-schema-tp
794cc1613801        hyperledger/sawtooth-validator:1.1             "bash -c '\n  if [ ! …"   34 seconds ago      Up 30 seconds       0.0.0.0:4020->4004/tcp             grid-sawtooth-validator
```

Checklist: Are the following Docker Containers running?

* hyperledger/sawtooth-settings-tp:1.1
* hyperledger/sawtooth-rest-api:1.1
* hyperledger/sawtooth-devmode-engine-rust:1.1
* grid-track-and-trace-tp
* grid_gridd
* postgres
* hyperledger/sawtooth-shell:1.1
* grid-pike-tp
* grid-schema-tp
* hyperledger/sawtooth-validator:1.1


### docker-compose-installed.yaml ###

```
docker-compose -f docker-compose-installed.yaml up
```

