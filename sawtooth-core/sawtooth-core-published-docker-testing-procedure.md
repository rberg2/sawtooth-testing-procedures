# Sawtooth-core Docker Published Compose Testing Procedure #

## Docker Published Images ##

The steps in this section use the Docker images published to Dockerhub running
released sawtooth-core code.

### docker/compose/sawtooth-default.yaml - Devmode Consensus ###

This docker-compose file will bring a single node Devmode network up using the
stable Sawtooth Docker Images published to Dockerhub.

```
$ docker-compose -f docker/compose/sawtooth-default.yaml up
```

In another terminal

```
$ docker-compose -f docker/compose/sawtooth-default.yaml exec shell bash
```

Run a few tests inside this docker container.

First we will show the block list, observing we have the Genesis block.

```
root@4704d99f36eb:/# sawtooth block list --url http://sawtooth-rest-api-default:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
0	9037e9dcd19a3b7edf5a17f98d6c4946d0f223f227d2059aae74232b1ee8d5ed53433ef410c15fea4b8bd206c146f6017cab8141c6ae12e9ada85fa1f2d8d293  2 	3 	03738ad5...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set and show a value.

```
root@4704d99f36eb:/# intkey set pineapples 5 --url http://sawtooth-rest-api-default:8008
{
  "link": "http://sawtooth-rest-api-default:8008/batch_statuses?id=f6156d54d9eb46906d0d3d6c0bebc6a5db1e6be79df241ee641967c44bf8df560c0d037683b3435a3ac47ccd264a28bd699a306f65a1c9e128094f02c84e90a7"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@4704d99f36eb:/# intkey create_batch
Writing to batches.intkey...
root@4704d99f36eb:/# intkey load --url http://rest-api:8008
batches: 2 batch/sec: 169.55931518201848
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@4704d99f36eb:/# intkey show pineapples --url http://sawtooth-rest-api-default:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@4704d99f36eb:/# intkey list --url http://sawtooth-rest-api-default:8008
pineapples: 5
KVOfyO: 61516
root@4704d99f36eb:/# sawtooth block list --url http://sawtooth-rest-api-default:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	a1aad75b675407c7177fe6201dd1c4607d8c8419e3c9af556ad1ca65b01174b73f3d36bbf9d3758051debb6af0bea0d3883f56f1a0a8b846c447791094b45c49  1 	1 	03738ad5...
1	d27f02e5e5937e04488e57ddb5115a9a6ee05533fa3e4dda880e92b96f47defb179b4b040d28136c30632267189a8ffc3ee86cb7ce55f77bb668d5d45ee83822  2 	3 	03738ad5...
0	9037e9dcd19a3b7edf5a17f98d6c4946d0f223f227d2059aae74232b1ee8d5ed53433ef410c15fea4b8bd206c146f6017cab8141c6ae12e9ada85fa1f2d8d293  2 	3 	03738ad5...
root@4704d99f36eb:/# exit
```

Checklist:

* Are there 2 intkey transactions
* Is pineapples equal to 5
* Are there 3 blocks on the chain

Control-C the first terminal where we created this network with docker-compose
and bring the network down to remove the containers.

```
$ docker-compose -f docker/compose/sawtooth-default.yaml down
```

Checklist: Check for this output after running the 'down' command

* Removing sawtooth-shell-default           	... done
* Removing sawtooth-settings-tp-default     	... done
* Removing sawtooth-rest-api-default        	... done
* Removing sawtooth-xo-tp-python-default    	... done
* Removing sawtooth-devmode-engine-rust-default ... done
* Removing sawtooth-intkey-tp-python-default	... done
* Removing sawtooth-validator-default       	... done
* Removing network compose_default

### docker/compose/sawtooth-default-pbft.yaml - PBFT Consensus ###

This docker-compose file will bring a five node PBFT network up using the
stable Sawtooth Docker Images published to Dockerhub.

```
$ docker-compose -f docker/compose/sawtooth-default-pbft.yaml up
```

Run a few tests inside this docker container.

First we will show the block list, observing we have the Genesis block.

```
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-0:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set a value.

```
root@edaed9f8a601:/# intkey set pineapples 5 --url http://rest-api-0:8008
{
  "link": "http://rest-api-0:8008/batch_statuses?id=bb026f53f01ad0ab16019969292ac2e5ba98c6536621a7fcb45e37c1ae70912f44d7e89e696f2da2379200f90567b64853ce9b6316d742431dadf2bf546ece1e"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@edaed9f8a601:/# intkey create_batch
Writing to batches.intkey...
root@edaed9f8a601:/# intkey load --url http://rest-api-0:8008
batches: 2 batch/sec: 74.29464175006642
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-0:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-0:8008
pineapples: 5
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-1:8008
pineapples: 5
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-2:8008
pineapples: 5
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-3:8008
pineapples: 5
root@edaed9f8a601:/# intkey show pineapples --url http://rest-api-4:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5 on every node

```
root@edaed9f8a601:/# intkey list --url http://rest-api-0:8008
pineapples: 5
kkygrF: 47313
root@edaed9f8a601:/# intkey list --url http://rest-api-1:8008
pineapples: 5
kkygrF: 47313
root@edaed9f8a601:/# intkey list --url http://rest-api-2:8008
pineapples: 5
kkygrF: 47313
root@edaed9f8a601:/# intkey list --url http://rest-api-3:8008
pineapples: 5
kkygrF: 47313
root@edaed9f8a601:/# intkey list --url http://rest-api-4:8008
pineapples: 5
kkygrF: 47313
```

Checklist:

* Are there 2 intkey transactions on every node

```
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-0:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	e7e08a022fda85d947defd7c36e892e04001ec2d5239a44e630988ad1c471bd4496d602deb27c220722f6523e34156deb22ed7a120c6de520dfd2e5f2b43a80c  2 	8 	0202ac1b...
1	53a75ea96e7d307bb3887fb30ef6ab721dabf8e2e127ea047436b286dd95fb780667eb942b4c45eff89a3992c695e2b93b52e5771d18f20cc5fdcfd4063d5245  1 	1 	0202ac1b...
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72...
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-1:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	e7e08a022fda85d947defd7c36e892e04001ec2d5239a44e630988ad1c471bd4496d602deb27c220722f6523e34156deb22ed7a120c6de520dfd2e5f2b43a80c  2 	8 	0202ac1b...
1	53a75ea96e7d307bb3887fb30ef6ab721dabf8e2e127ea047436b286dd95fb780667eb942b4c45eff89a3992c695e2b93b52e5771d18f20cc5fdcfd4063d5245  1 	1 	0202ac1b...
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72...
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-2:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	e7e08a022fda85d947defd7c36e892e04001ec2d5239a44e630988ad1c471bd4496d602deb27c220722f6523e34156deb22ed7a120c6de520dfd2e5f2b43a80c  2 	8 	0202ac1b...
1	53a75ea96e7d307bb3887fb30ef6ab721dabf8e2e127ea047436b286dd95fb780667eb942b4c45eff89a3992c695e2b93b52e5771d18f20cc5fdcfd4063d5245  1 	1 	0202ac1b...
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72...
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-3:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	e7e08a022fda85d947defd7c36e892e04001ec2d5239a44e630988ad1c471bd4496d602deb27c220722f6523e34156deb22ed7a120c6de520dfd2e5f2b43a80c  2 	8 	0202ac1b...
1	53a75ea96e7d307bb3887fb30ef6ab721dabf8e2e127ea047436b286dd95fb780667eb942b4c45eff89a3992c695e2b93b52e5771d18f20cc5fdcfd4063d5245  1 	1 	0202ac1b...
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72...
root@edaed9f8a601:/# sawtooth block list --url http://rest-api-4:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER  
2	e7e08a022fda85d947defd7c36e892e04001ec2d5239a44e630988ad1c471bd4496d602deb27c220722f6523e34156deb22ed7a120c6de520dfd2e5f2b43a80c  2 	8 	0202ac1b...
1	53a75ea96e7d307bb3887fb30ef6ab721dabf8e2e127ea047436b286dd95fb780667eb942b4c45eff89a3992c695e2b93b52e5771d18f20cc5fdcfd4063d5245  1 	1 	0202ac1b...
0	4d261ec6e393486109ead0b31b25a677a8459ea39a6c1b9501b0c91521d617e1764e1f308d4b7ed5170e04ad79cbf98e5b51efeff361ee450bb3023a91fa48b8  2 	5 	021b4e72…
```

Checklist:

* Are there 3 blocks on the chain for every node

```
root@edaed9f8a601:/# sawtooth peer list --url http://rest-api-0:8008
tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-3:8800,tcp://validator-4:8800
root@edaed9f8a601:/# sawtooth peer list --url http://rest-api-1:8008
tcp://validator-0:8800,tcp://validator-2:8800,tcp://validator-3:8800,tcp://validator-4:8800
root@edaed9f8a601:/# sawtooth peer list --url http://rest-api-2:8008
tcp://validator-0:8800,tcp://validator-1:8800,tcp://validator-3:8800,tcp://validator-4:8800
root@edaed9f8a601:/# sawtooth peer list --url http://rest-api-3:8008
tcp://validator-0:8800,tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-4:8800
root@edaed9f8a601:/# sawtooth peer list --url http://rest-api-4:8008
tcp://validator-0:8800,tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-3:8800
root@edaed9f8a601:/# exit
```

Checklist:

* Is every node peered with every other node?

Control-C the first terminal where we created this network with docker-compose
and bring the network down to remove the containers.

```
$ docker-compose -f docker/compose/sawtooth-default-pbft.yaml down
```

Checklist: Check for this output after running the 'down' command

* Removing sawtooth-pbft-engine-default-1  	... done
* Removing sawtooth-rest-api-default-3     	... done
* Removing sawtooth-rest-api-default-2     	... done
* Removing sawtooth-validator-default-3    	... done
* Removing sawtooth-xo-tp-python-default-3 	... done
* Removing sawtooth-pbft-engine-default-4  	... done
* Removing sawtooth-intkey-tp-python-default-0 ... done
* Removing sawtooth-xo-tp-python-default-2 	... done
* Removing sawtooth-pbft-engine-default-3  	... done
* Removing sawtooth-validator-default-0    	... done
* Removing sawtooth-settings-tp-default-1  	... done
* Removing sawtooth-xo-tp-python-default-4 	... done
* Removing sawtooth-settings-tp-default-4  	... done
* Removing sawtooth-xo-tp-python-default-0 	... done
* Removing sawtooth-intkey-tp-python-default-3 ... done
* Removing sawtooth-rest-api-default-0     	... done
* Removing sawtooth-intkey-tp-python-default-2 ... done
* Removing sawtooth-rest-api-default-4     	... done
* Removing sawtooth-pbft-engine-default-0  	... done
* Removing sawtooth-intkey-tp-python-default-1 ... done
* Removing sawtooth-shell-default          	... done
* Removing sawtooth-settings-tp-default-3  	... done
* Removing sawtooth-validator-default-2    	... done
* Removing sawtooth-rest-api-default-1     	... done
* Removing sawtooth-settings-tp-default-0  	... done
* Removing sawtooth-validator-default-1    	... done
* Removing sawtooth-settings-tp-default-2  	... done
* Removing sawtooth-xo-tp-python-default-1 	... done
* Removing sawtooth-validator-default-4    	... done
* Removing sawtooth-intkey-tp-python-default-4 ... done
* Removing sawtooth-pbft-engine-default-2  	... done
* Removing network compose_default

### docker/compose/sawtooth-default-poet.yaml - POET Consensus ###

This docker-compose file will bring a five node POET network up using the
stable Sawtooth Docker Images published to Dockerhub.

```
$ docker-compose -f docker/compose/sawtooth-default-poet.yaml up
```

In another terminal

```
$ docker-compose -f docker/compose/sawtooth-default-poet.yaml exec shell bash
```

Run a few tests inside this docker container.

First we will show the block list, observing we have the Genesis and settings
blocks.

```
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-0:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
```

Checklist:

* Are blocks 0 (genesis) and 1 present

Next we will use intkey to set a value.

```
root@7cfe4b71158c:/# intkey set pineapples 5 --url http://rest-api-0:8008
{
  "link": "http://rest-api-0:8008/batch_statuses?id=9ed00b9f7a719a02063f1ef5bdaf6d3a5c44fe1a63f8977e7fb4c2c5620a5ccd6e184b5bf577325098db8990491e27ba68e060e99db9d75b4f94613dda00ddad"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@7cfe4b71158c:/# intkey create_batch
Writing to batches.intkey...
root@7cfe4b71158c:/# intkey load --url http://rest-api-0:8008
batches: 2 batch/sec: 105.12698790651044
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-0:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-0:8008
pineapples: 5
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-1:8008
pineapples: 5
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-2:8008
pineapples: 5
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-3:8008
pineapples: 5
root@7cfe4b71158c:/# intkey show pineapples --url http://rest-api-4:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5 on every node

```
root@7cfe4b71158c:/# intkey list --url http://rest-api-0:8008
ATeaXb: 51934
pineapples: 5
root@7cfe4b71158c:/# intkey list --url http://rest-api-1:8008
ATeaXb: 51934
pineapples: 5
root@7cfe4b71158c:/# intkey list --url http://rest-api-2:8008
ATeaXb: 51934
pineapples: 5
root@7cfe4b71158c:/# intkey list --url http://rest-api-3:8008
ATeaXb: 51934
pineapples: 5
root@7cfe4b71158c:/# intkey list --url http://rest-api-4:8008
ATeaXb: 51934
pineapples: 5
```

Checklist:

* Are there 2 intkey transactions on every node

```
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-0:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
3	099d06f882d349b0c65d37104734ede8e3ea3883ca05944bbf73cd328b3e600854560774bf540b01c4ca841d76e2709324962a609bf426875d07136c3760c7b3  2 	5 	03a6cc1d1d81317...
2	bc42ff32f23a2d145fdf314bef84b7eda142f9854bbc089077f96b529b1534a12acd41d9a548c1e2601cafc1b8f7327af44b58794364bd0aa8f23448d902fa88  1 	1 	02a7e369b5e6615...
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-1:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
3	099d06f882d349b0c65d37104734ede8e3ea3883ca05944bbf73cd328b3e600854560774bf540b01c4ca841d76e2709324962a609bf426875d07136c3760c7b3  2 	5 	03a6cc1d1d81317...
2	bc42ff32f23a2d145fdf314bef84b7eda142f9854bbc089077f96b529b1534a12acd41d9a548c1e2601cafc1b8f7327af44b58794364bd0aa8f23448d902fa88  1 	1 	02a7e369b5e6615...
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-2:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
3	099d06f882d349b0c65d37104734ede8e3ea3883ca05944bbf73cd328b3e600854560774bf540b01c4ca841d76e2709324962a609bf426875d07136c3760c7b3  2 	5 	03a6cc1d1d81317...
2	bc42ff32f23a2d145fdf314bef84b7eda142f9854bbc089077f96b529b1534a12acd41d9a548c1e2601cafc1b8f7327af44b58794364bd0aa8f23448d902fa88  1 	1 	02a7e369b5e6615...
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-3:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
3	099d06f882d349b0c65d37104734ede8e3ea3883ca05944bbf73cd328b3e600854560774bf540b01c4ca841d76e2709324962a609bf426875d07136c3760c7b3  2 	5 	03a6cc1d1d81317...
2	bc42ff32f23a2d145fdf314bef84b7eda142f9854bbc089077f96b529b1534a12acd41d9a548c1e2601cafc1b8f7327af44b58794364bd0aa8f23448d902fa88  1 	1 	02a7e369b5e6615...
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
root@7cfe4b71158c:/# sawtooth block list --url http://rest-api-4:8008
NUM  BLOCK_ID                                                                                                                      	BATS  TXNS  SIGNER    	 
3	099d06f882d349b0c65d37104734ede8e3ea3883ca05944bbf73cd328b3e600854560774bf540b01c4ca841d76e2709324962a609bf426875d07136c3760c7b3  2 	5 	03a6cc1d1d81317...
2	bc42ff32f23a2d145fdf314bef84b7eda142f9854bbc089077f96b529b1534a12acd41d9a548c1e2601cafc1b8f7327af44b58794364bd0aa8f23448d902fa88  1 	1 	02a7e369b5e6615...
1	991e62c1944b0958cdcee5c1d7f2d844e3829747d479b5764cd3337659529e9b6a05e0054f0152cd8921e7f04363d4b3d4bd1c52d85361ed3f981d2dcc30cbb0  4 	4 	0277f18be32afca...
0	c52f0aa798b3600ce65db6878275fe18ca418b42d81eeece429a1125679115222cd27b23e21164589264c1d4a7bb4fe5ee19c564259b2e74616d80688cf270ee  4 	10	0277f18be32afca...
```

Checklist:

* Are there 4 blocks on the chain for every node

```
root@7cfe4b71158c:/# sawtooth peer list --url http://rest-api-0:8008
tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-3:8800
root@7cfe4b71158c:/# sawtooth peer list --url http://rest-api-1:8008
tcp://validator-0:8800,tcp://validator-2:8800,tcp://validator-4:8800
root@7cfe4b71158c:/# sawtooth peer list --url http://rest-api-2:8008
tcp://validator-0:8800,tcp://validator-1:8800,tcp://validator-3:8800,tcp://validator-4:8800
root@7cfe4b71158c:/# sawtooth peer list --url http://rest-api-3:8008
tcp://validator-0:8800,tcp://validator-2:8800,tcp://validator-4:8800
root@7cfe4b71158c:/# sawtooth peer list --url http://rest-api-4:8008
tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-3:8800
root@7cfe4b71158c:/# exit
```

Checklist:

* Is every node peered with other nodes and none are excluded

Control-C the first terminal where we created this network with docker-compose
and bring the network down to remove the containers.

```
$ docker-compose -f docker/compose/sawtooth-default-poet.yaml down
```

Checklist: Check for this output after running the 'down' command

* Removing sawtooth-xo-tp-python-default-1   	... done
* Removing sawtooth-xo-tp-python-default-2   	... done
* Removing sawtooth-poet-validator-registry-tp-4 ... done
* Removing sawtooth-poet-validator-registry-tp-3 ... done
* Removing sawtooth-shell-default            	... done
* Removing sawtooth-settings-tp-default-1    	... done
* Removing sawtooth-poet-engine-1            	... done
* Removing sawtooth-intkey-tp-python-default-3   ... done
* Removing sawtooth-validator-default-1      	... done
* Removing sawtooth-intkey-tp-python-default-0   ... done
* Removing sawtooth-poet-engine-2            	... done
* Removing sawtooth-poet-engine-0            	... done
* Removing sawtooth-rest-api-default-0       	... done
* Removing sawtooth-rest-api-default-2       	... done
* Removing sawtooth-xo-tp-python-default-3   	... done
* Removing sawtooth-poet-validator-registry-tp-0 ... done
* Removing sawtooth-intkey-tp-python-default-1   ... done
* Removing sawtooth-poet-validator-registry-tp-2 ... done
* Removing sawtooth-settings-tp-default-4    	... done
* Removing sawtooth-validator-default-3      	... done
* Removing sawtooth-validator-default-4      	... done
* Removing sawtooth-validator-default-0      	... done
* Removing sawtooth-intkey-tp-python-default-4   ... done
* Removing sawtooth-settings-tp-default-2    	... done
* Removing sawtooth-xo-tp-python-default-4   	... done
* Removing sawtooth-rest-api-default-4       	... done
* Removing sawtooth-rest-api-default-3       	... done
* Removing sawtooth-intkey-tp-python-default-2   ... done
* Removing sawtooth-rest-api-default-1       	... done
* Removing sawtooth-settings-tp-default-3    	... done
* Removing sawtooth-poet-validator-registry-tp-1 ... done
* Removing sawtooth-validator-default-2      	... done
* Removing sawtooth-xo-tp-python-default-0   	... done
* Removing sawtooth-poet-engine-4            	... done
* Removing sawtooth-settings-tp-default-0    	... done
* Removing sawtooth-poet-engine-3            	... done
* Removing network compose_default

### Final ###

We have now run sawtooth from the published Docker Images and verified the
functionality with the provided demo docker-compose files.
