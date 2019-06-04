# Sawtooth-core Testing Procedure #

## Building code and Docker Images ##

The steps in this section compile and package the sawtooth-core code and builds
new Docker images with this developmental code.

### Build local Docker Images ###

First lets build the local Docker Images used to run the compiled code passed
in as a volume.

```
$ cd sawtooth-core
$ docker-compose -f docker/compose/sawtooth-build.yaml up
```

### Verification Step ###

To verify this step has been completed correctly run the following command and
ensure all "-local" images have been built and have a recent CREATED time.

```
$ docker images | grep local
REPOSITORY               	TAG             	IMAGE ID        	CREATED         	SIZE
identity-rust-tp-local   	latest          	731fbd7ee0bd    	2 minutes ago   	1.24GB
block-info-tp-local      	latest          	fb210982da41    	3 minutes ago   	1.24GB
sawtooth-rest-api-local  	latest          	fe69054d5d78    	4 minutes ago   	502MB
sawtooth-cli-local       	latest          	e6056a6349c1    	4 minutes ago   	498MB
smallbank-workload-local 	latest          	9e63bb4ed2cc    	5 minutes ago   	1.24GB
smallbank-rust-tp-local  	latest          	7476746cab3c    	5 minutes ago   	1.24GB
sawtooth-settings-tp-local	latest          	3427d579e7dd    	6 minutes ago   	1.24GB
sawtooth-adm-local       	latest          	fb99d12c8d89    	6 minutes ago   	1.24GB
sawtooth-validator-local 	latest          	508ff87b476f    	8 minutes ago   	1.44GB
```

Checklist: Have the following Docker images been created?

* identity-rust-tp-local
* block-info-tp-local
* sawtooth-rest-api-local
* sawtooth-cli-local
* smallbank-workload-local
* smallbank-rust-tp-local
* sawtooth-settings-tp-local
* sawtooth-adm-local
* sawtooth-validator-local

### Build Debs and Docker Images with Debs installed ###

Now build the installed Docker Images that contain the installed sawtooth deb
packages.

```
$ docker-compose build
$ docker-compose -f docker-compose-installed.yaml build
```

#### Verification Step ####

Run this command and verify these Docker images were created in the proceeding
step. We will be testing the actual code in the next steps.

```
$ docker images
REPOSITORY                	TAG             	IMAGE ID        	CREATED         	SIZE
sawtooth-meta             	latest          	7f63c26a4a89    	About an hour ago	322MB
<none>                    	<none>          	eb1e5518c1d5    	About an hour ago	1.89GB
sawtooth-intkey-workload  	latest          	d529d9ee6114    	About an hour ago	140MB
<none>                    	<none>          	8d1fb00bff77    	About an hour ago	3.12GB
sawtooth-smallbank-workload	latest          	7c3972c10338    	About an hour ago	192MB
<none>                    	<none>          	49697456668b    	About an hour ago	3.15GB
<none>                    	<none>          	4a656ff6efeb    	About an hour ago	2.06GB
sawtooth-integration      	latest          	b5d1651f4334    	About an hour ago	173MB
<none>                    	<none>          	e4997b3d0a7d    	About an hour ago	1.89GB
sawtooth-identity-tp      	latest          	7ef02c24cfa3    	About an hour ago	237MB
<none>                    	<none>          	61a808a131bf    	About an hour ago	2.96GB
sawtooth-block-info-tp    	latest          	db18d587d30f    	About an hour ago	236MB
<none>                    	<none>          	4ec5dbc1087a    	About an hour ago	2.95GB
sawtooth-cli              	latest          	c4b5bce86630    	About an hour ago	145MB
sawtooth-admin-tools      	latest          	e2a0d1c71b58    	About an hour ago	138MB
<none>                    	<none>          	d79a8ed81a36    	About an hour ago	3.08GB
sawtooth-shell            	latest          	880bac34564f    	2 hours ago			330MB
```

Checklist: Have the following Docker images been created?

* sawtooth-meta
* sawtooth-intkey-workload
* sawtooth-smallbank-workload
* sawtooth-integration
* sawtooth-identity-tp
* sawtooth-block-info-tp
* sawtooth-cli
* sawtooth-admin-tools
* sawtooth-shell

### docker-compose.yaml ###

After the Sawtooth-core code and Docker images have been built, 'up' the
docker-compose.yaml file and exec into the client container. To test the
compiled code passed in as a volume, run a couple of demo commands to create
transactions. Next check the transactions are there and the block list has
grown.

```
$ docker-compose up
```

In another terminal enter the client Docker Container.

```
$ docker-compose exec client bash
```

Run a few test commands inside this Docker container.

First we will show the block list, observing we have the Genesis block.

```
root@e3761751d372:/project/sawtooth-core# sawtooth block list --url http://rest-api:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  

0	ab7324f431e0aae908004e17531e3e28b5f65bdcb27387ac081ba403fd1944f679362bc4679b8664d2c3a7c42806dc07f05ef4b0986128b58399d3e89fa9d366  2 	3 	028b81d1...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set and show a value.

```
root@e3761751d372:/project/sawtooth-core# intkey set pineapples 5 --url http://rest-api:8008
{
  "link": "http://rest-api:8008/batch_statuses?id=bd5182c98aefa049118045ae360d84ba9570b7dccd9907ad60b8038a62b8129522718eec79a6c4f242740764f5364ee0129e3dd1a86181a108a68a8dafa5c626"
}
```

Checklist:

* Did this set command exit without error

Next create a batch with increment and decrement transactions then load it.

```
root@e3761751d372:/project/sawtooth-core# intkey create_batch
Writing to batches.intkey...
root@e3761751d372:/project/sawtooth-core# intkey load --url http://rest-api:8008
batches: 2 batch/sec: 107.3838039888374
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@e3761751d372:/project/sawtooth-core# intkey show pineapples --url http://rest-api:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@e3761751d372:/project/sawtooth-core# intkey list --url http://rest-api:8008
xDEoYt: 49947
pineapples: 5

root@e3761751d372:/project/sawtooth-core# sawtooth block list --url http://rest-api:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	c4e64aaae29accac560a8803d3808dd32755cfe67170cf4592090a684a2538535fe8937942e575b7cf513d70a728843a5b508daff3933860ca49da9f20e819ec  2 	11	028b81d1...
1	4345febb31f09bd68fe0833333ea56d16ee4748e8b1a47e7d7c35ec00b2fc31a2e57395898aee3db6fb6716888a23afbff44d09bc4723cf868b8ffc80e90619d  1 	1 	028b81d1...
0	ab7324f431e0aae908004e17531e3e28b5f65bdcb27387ac081ba403fd1944f679362bc4679b8664d2c3a7c42806dc07f05ef4b0986128b58399d3e89fa9d366  2 	3 	028b81d1…
root@e3761751d372:/project/sawtooth-core# exit
```

Checklist:

* Are there 2 intkey transactions
* Is pineapples equal to 5
* Are there 3 blocks on the chain

Control-C the first terminal where we created this network with docker-compose
and bring the network down to remove the containers.

```
$ docker-compose down
```

Checklist: Check for this output after running the 'down' command

* Removing sawtooth-devmode-engine-rust	... done
* Removing sawtooth-rest-api-local		... done
* Removing sawtooth-settings-tp-local	... done
* Removing sawtooth-intkey-tp-python	... done
* Removing sawtooth-xo-tp-python		... done
* Removing sawtooth-shell-local			... done
* Removing sawtooth-validator-local		... done
* Removing network sawtooth-core_default

### docker-compose-installed.yaml ###

This docker-compose file will bring the network up using the compiled debs
installed inside Docker Images

```
$ docker-compose -f docker-compose-installed.yaml up
```

In another terminal.

```
$ docker-compose -f docker-compose-installed.yaml exec shell bash
```

Run a few test commands inside this Docker container.

First we will show the block list, observing we have the Genesis block.

```
root@e9f05abe4e36:/# sawtooth block list --url http://rest-api:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
0	02765c6ee79c14639e8ac20b130b52063575f1aa6678cfae9633383a86accd2928af05da13b7966572ef0d8f6e323205c80cacb700360a65bec347f6b0b72c34  2 	3 	0246a931...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set and show a value.

```
root@e9f05abe4e36:/# intkey set pineapples 5 --url http://rest-api:8008
{
  "link": "http://rest-api:8008/batch_statuses?id=264cde60d95be00a64c8954898a0ce25edf559c75f5459612607662cb28ac8c63209b136dfe14590a72c22525f7fc7b76c96ffea2cdc13bf8469d4cce916d248"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@e9f05abe4e36:/# intkey create_batch
Writing to batches.intkey...
root@e9f05abe4e36:/# intkey load --url http://rest-api:8008
batches: 2 batch/sec: 111.11033404858406
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@e9f05abe4e36:/# intkey show pineapples --url http://rest-api:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@e9f05abe4e36:/# intkey list --url http://rest-api:8008
pineapples: 5
mKMzbU: 68951
root@e9f05abe4e36:/# sawtooth block list --url http://rest-api:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	9555e924a775c3a6c8f4ac6c36e9e71688425650e24e12f8ced0c8349f344faf574c3f88ad599db1300186bc2d23e7192fdc67886c87489f8e731cd2cb9e6d79  2 	10	0246a931...
1	b10ba23c50cd988b32cbac3c011d428cdfdaad13774b73611a3bcf260c972cf40d81e6dee70fb811155e1bcb76545d314d45ff741c6ce5f8bb92e6bdd51bd22c  1 	1 	0246a931...
0	02765c6ee79c14639e8ac20b130b52063575f1aa6678cfae9633383a86accd2928af05da13b7966572ef0d8f6e323205c80cacb700360a65bec347f6b0b72c34  2 	3 	0246a931...
root@e9f05abe4e36:/# exit
```

Checklist:

* Are there 2 intkey transactions
* Is pineapples equal to 5
* Are there 3 blocks on the chain

Control-C the first terminal where we created this network with docker-compose
and bring the network down to remove the containers.

```
$ docker-compose -f docker-compose-installed.yaml down
```

Checklist: Check for this output after running the 'down' command

* Removing sawtooth-shell-default        	... done
* Removing sawtooth-intkey-tp-python-default ... done
* Removing sawtooth-settings-tp-default  	... done
* Removing sawtooth-rest-api-default     	... done
* Removing sawtooth-xo-tp-python-default 	... done
* Removing sawtooth-devmode-engine-rust  	... done
* Removing sawtooth-identity-tp-default  	... done
* Removing sawtooth-intkey-workload      	... done
* Removing sawtooth-smallbank-workload   	... done
* Removing sawtooth-meta                 	            ... done
* Removing sawtooth-block-info-tp-default	... done
* Removing sawtooth-admin-tools          	... done
* Removing sawtooth-validator-default    	... done
* Removing sawtooth-cli-default            	... done
* Removing sawtooth-integration-default  	... done
* Removing network sawtooth-core_default

### docker/compose/copy-debs.yaml ###

This docker-compose file copies the compiled deb packages out of the images
created by docker-compose-installed.yaml to sawtooth-core/build/debs

```
$ docker-compose -f docker/compose/copy-debs.yaml up
```

#### Verification Step ####

List files in build/debs and verify that they are present and have a recent
timestamp.

```
ls -l build/debs/
total 9256
-rw-r--r-- 1 root root   52752 May 15 16:46 python3-sawtooth-cli_1.2.1-dev531-dirty-1_all.deb
-rw-r--r-- 1 root root   21984 May 15 16:46 python3-sawtooth-integration_1.2.1-dev531-dirty-1_all.deb
-rw-r--r-- 1 root root   45280 May 15 16:46 python3-sawtooth-rest-api_1.2.1-dev531-dirty-1_all.deb
-rw-r--r-- 1 root root 4941700 May 15 16:46 python3-sawtooth-validator_1.2.1-dev531-dirty-1_all.deb
-rw-r--r-- 1 root root  438892 May 15 16:46 sawadm_1.2.1-dev531-dirty_amd64.deb
-rw-r--r-- 1 root root    2264 May 15 16:46 sawtooth_1.2.1-dev531-dirty_all.deb
-rw-r--r-- 1 root root  572702 May 15 16:46 sawtooth-block-info-tp_1.2.1-dev531-dirty_amd64.deb
-rw-r--r-- 1 root root  612240 May 15 16:46 sawtooth-identity-tp_1.2.1-dev531-dirty_amd64.deb
-rw-r--r-- 1 root root  897742 May 15 16:46 sawtooth-intkey-workload_1.2.1-dev531-dirty_amd64.deb
-rw-r--r-- 1 root root  602880 May 15 16:46 sawtooth-settings-tp_1.2.1-dev531-dirty_amd64.deb
-rw-r--r-- 1 root root 1263886 May 15 16:46 sawtooth-smallbank-workload_1.2.1-dev531-dirty_amd64.deb
```

Checklist: Have the following Debian packages been created in build/debs

* python3-sawtooth-cli_1.2.1-dev531-dirty-1_all.deb
* python3-sawtooth-integration_1.2.1-dev531-dirty-1_all.deb
* python3-sawtooth-rest-api_1.2.1-dev531-dirty-1_all.deb
* python3-sawtooth-validator_1.2.1-dev531-dirty-1_all.deb
* sawadm_1.2.1-dev531-dirty_amd64.deb
* sawtooth_1.2.1-dev531-dirty_all.deb
* sawtooth-block-info-tp_1.2.1-dev531-dirty_amd64.deb
* sawtooth-identity-tp_1.2.1-dev531-dirty_amd64.deb
* sawtooth-intkey-workload_1.2.1-dev531-dirty_amd64.deb
* sawtooth-settings-tp_1.2.1-dev531-dirty_amd64.deb
* sawtooth-smallbank-workload_1.2.1-dev531-dirty_amd64.deb

```
docker-compose -f docker/compose/copy-debs.yaml down
```

Checklist: Check for this output after running the 'down' command

* Removing compose_intkey-workload_1_8a6f9a465084    ... done
* Removing compose_sawtooth-meta_1_da22ff28adff      ... done
* Removing compose_integration_1_c617e4f076d6        ... done
* Removing compose_sawtooth-cli_1_d6e4139f0801       ... done
* Removing compose_rest-api_1_5ae5f6caea45           ... done
* Removing compose_smallbank-workload_1_9e79760ddc88 ... done
* Removing compose_settings-tp_1_ce9dea63392b        ... done
* Removing compose_shell_1_81e292c462bd              ... done
* Removing compose_block-info-tp_1_802641fc1986      ... done
* Removing compose_identity-tp_1_bf5a8e755492        ... done
* Removing compose_validator_1_cd4be7817e1a          ... done
* Removing compose_admin-tools_1_e7e743817298        ... done
* Removing network compose_default

### Final ###

We have now built and run sawtooth from source code and verified this works
with the demo docker-compose networks.
