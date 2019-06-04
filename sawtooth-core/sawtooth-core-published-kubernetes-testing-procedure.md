# Sawtooth-core Docker Published Kubernetes Testing Procedure #

## Kubernetes Networks ##

This document outlines a method of testing the released sawtooth-core code
published to Dockerhub with Kubernetes.
The steps in this document use the Docker images published to Dockerhub running
released sawtooth-core code to start a network in Kubernetes.

### Kubernetes - Devmode Consensus ###

This yaml file will create a single sawtooth pod running Devmode consensus on a
Kubernetes cluster using the published Dockerhub images.

First create the network.

```
$ kubectl apply -f docker/kubernetes/sawtooth-kubernetes-default.yaml
deployment.extensions/sawtooth-0 created
service/sawtooth-0 created
```

Get the pod name.

```
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
sawtooth-0-79f54645f-wbfdw   7/7     Running   1          4m1s
```

Checklist:

* are 7 of 7 Containers ready in this pod

Enter the shell container in the pod.

```
$ kubectl exec -it sawtooth-0-79f54645f-wbfdw --container sawtooth-shell -- bash
root@sawtooth-0-79f54645f-wbfdw:/#
```

Run a few tests inside this container.

First we will show the block list, observing we have the Genesis block.

```
root@sawtooth-0-79f54645f-wbfdw:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
0    cc08dd532b34f29df6e7dd61319f2906694a71ed8aa100787e238dd9ec53404962bcd0d82a923c78bbe8ac0c9328e654513d3c1a749d93ce381cd5da8e150eba  2     3     039a3c10...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set and show a value.

```
root@sawtooth-0-79f54645f-wbfdw:/# intkey set pineapples 5
{
  "link": "http://127.0.0.1:8008/batch_statuses?id=2a5e22667bee90c66415e407fb7bae3303c3cd2f3dda6bce188475f95a8816bb4f3a23d3786860fa96dff2fc21d9089af1d04e1491849aba1b50ce84f66b7da2"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@sawtooth-0-79f54645f-wbfdw:/# intkey create_batch
Writing to batches.intkey...
root@sawtooth-0-79f54645f-wbfdw:/# intkey load
batches: 2 batch/sec: 193.2547285000115
```

#### Verification Step ####

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@sawtooth-0-79f54645f-wbfdw:/# intkey show pineapples
pineapples: 5
```

Checklist:

* Is pineapples equal to 5

```
root@sawtooth-0-79f54645f-wbfdw:/# intkey list
Yqwuau: 21024
pineapples: 5
root@sawtooth-0-79f54645f-wbfdw:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
2    799bbe7fd64f638bc41be212ffa9108e47141b4fc6ad8c1b0e5c75e62d06398535c28bed4284260a4300e9b8cc7e821f6bcacbac9fd8ba3da0225931f003ac87  2     10    039a3c10...
1    04d9ae2e41015c5430e0a58c0d31ab8cb89c91486345f8f7f8ceadc508c534c00182af40fb6dc715bb90019d5a9825fd7ece9e4d781be55e6812ae1f164f3101  1     1     039a3c10...
0    cc08dd532b34f29df6e7dd61319f2906694a71ed8aa100787e238dd9ec53404962bcd0d82a923c78bbe8ac0c9328e654513d3c1a749d93ce381cd5da8e150eba  2     3     039a3c10...
root@sawtooth-0-79f54645f-wbfdw:/# exit
```

Checklist:

* Are there 2 intkey transactions
* Is pineapples equal to 5
* Are there 3 blocks on the chain

Now delete the Sawtooth network.

```
$ kubectl delete -f docker/kubernetes/sawtooth-kubernetes-default.yaml
```

Checklist: Check for this output after deleting sawtooth-kubernetes-default

* deployment.extensions "sawtooth-0" deleted
* service "sawtooth-0" deleted

### Kubernetes - PBFT Consensus ###

These yaml files will create a five pod Sawtooth network running the PBFT
consensus engine on an Kubernetes cluster using the published Dockerhub images.

First pre-generate keypairs.

```
$ kubectl apply -f docker/kubernetes/sawtooth-create-pbft-keys.yaml
job.batch/pbft-keys created
```

Retrieve the logs from this pod and make a note of the keys

```
$ kubectl log pbft-keys-wfdq2
pbft0priv: 028de9ced7ae7c58f1c4b8bb84a8cbf9378eb5943948d2dd6f493d0e7f3cadf3
pbft0pub: 036c14100c00188f0fe8e686d577f74f35032f97f10d78764f3ec910472f157c15
pbft1priv: c3e91eac9f8ccebc4d25977b69dc7137e4cf5fc6356a79c36802e712a49fd4b7
pbft1pub: 02554eddb36a2d0abcb7b51cd2316b213ebe030328cd949ebc105d061e8b6de5b5
pbft2priv: d34d4133fc830556beb8c642f31db4f082773c557b4108ec409871a10d60d2f4
pbft2pub: 03a86840dc802e19ea64b130d26f1ab7100f923396d3151ddedc76560bbdf2f778
pbft3priv: b8c8c79f7c8fe89eae18a805a836e23ad415b014822c60ac244a14c4df2e9429
pbft3pub: 02a87d1a87ed54cd467d1628a601e780ddc70a6718e4b98407f3d70bb88e8a1ee0
pbft4priv: 6499688038b519bfc99564978918ce31d74aa758380852608481c7cc1f779483
pbft4pub: 03186440394f4a447509874095a14550bc1b1821555560f0851a5f3549c8138ac2
```

Checklist:

* are there 5 pub / priv key pairs

Edit the pbft-keys-configmap YAML and add the keys from the step above.

```
vim docker/kubernetes/pbft-keys-configmap.yaml
```

With the end result looking like this.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: keys-config
data:
  pbft0priv: 028de9ced7ae7c58f1c4b8bb84a8cbf9378eb5943948d2dd6f493d0e7f3cadf3
  pbft0pub: 036c14100c00188f0fe8e686d577f74f35032f97f10d78764f3ec910472f157c15
  pbft1priv: c3e91eac9f8ccebc4d25977b69dc7137e4cf5fc6356a79c36802e712a49fd4b7
  pbft1pub: 02554eddb36a2d0abcb7b51cd2316b213ebe030328cd949ebc105d061e8b6de5b5
  pbft2priv: d34d4133fc830556beb8c642f31db4f082773c557b4108ec409871a10d60d2f4
  pbft2pub: 03a86840dc802e19ea64b130d26f1ab7100f923396d3151ddedc76560bbdf2f778
  pbft3priv: b8c8c79f7c8fe89eae18a805a836e23ad415b014822c60ac244a14c4df2e9429
  pbft3pub: 02a87d1a87ed54cd467d1628a601e780ddc70a6718e4b98407f3d70bb88e8a1ee0
  pbft4priv: 6499688038b519bfc99564978918ce31d74aa758380852608481c7cc1f779483
  pbft4pub: 03186440394f4a447509874095a14550bc1b1821555560f0851a5f3549c8138ac2
```

Apply this configmap so that the sawtooth network can use these keys.

```
$ kubectl apply -f docker/kubernetes/pbft-keys-configmap.yaml
configmap/keys-config created
```

Next create the network.

```
$ kubectl apply -f docker/kubernetes/sawtooth-kubernetes-default-pbft.yaml
deployment.extensions/pbft-0 created
service/sawtooth-0 created
deployment.extensions/pbft-1 created
service/sawtooth-1 created
deployment.extensions/pbft-2 created
service/sawtooth-2 created
deployment.extensions/pbft-3 created
service/sawtooth-3 created
deployment.extensions/pbft-4 created
service/sawtooth-4 created
```

Get the pod names.

```
$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
pbft-0-6665c5795b-snhqh   8/8     Running     0          98s
pbft-1-665dd6d54b-jmzs5   8/8     Running     0          98s
pbft-2-6d8fd677dd-hz4qt   8/8     Running     0          97s
pbft-3-6668bfd599-z4q7x   8/8     Running     0          97s
pbft-4-9f45f6ddc-sfftv    8/8     Running     0          97s
pbft-keys-wfdq2           0/1     Completed   0          60m
```

Checklist:

* Are there 5 running pods
* Do the 5 running pods have 8 ready containers each
* Do the 5 running pods have 0 restarts

Enter the shell container in a pod.

```
$ kubectl exec -it pbft-0-6665c5795b-snhqh --container sawtooth-shell -- bash
root@pbft-0-6665c5795b-snhqh:/#
```

Run a few tests inside this container.

First we will show the block list, observing we have the Genesis block.

```
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
```

Checklist:

* Is block 0 (genesis) present

Next we will use intkey to set a value.

```
root@pbft-0-6665c5795b-snhqh:/# intkey set pineapples 5
{
  "link": "http://127.0.0.1:8008/batch_statuses?id=26e53569b4519c406dab1258cbfc47f6724ea3cf3b7a8570cd116a916952e6ca55e764cacb697c6ad41e360753ce732ad1908cc33bc3240824b19d8c393a1e49"
}
```

Checklist:

* Did this set command exit without error

Generates a batch with increment and decrement transactions then load it.

```
root@pbft-0-6665c5795b-snhqh:/# intkey create_batch
Writing to batches.intkey...
root@pbft-0-6665c5795b-snhqh:/# intkey load
batches: 2 batch/sec: 168.21287773967796
```

### Verification Step ###

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@pbft-0-6665c5795b-snhqh:/# intkey show pineapples
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey show pineapples --url http://$SAWTOOTH_1_SERVICE_HOST:8008
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey show pineapples --url http://$SAWTOOTH_2_SERVICE_HOST:8008
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey show pineapples --url http://$SAWTOOTH_3_SERVICE_HOST:8008
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey show pineapples --url http://$SAWTOOTH_4_SERVICE_HOST:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5 on every node

```
root@pbft-0-6665c5795b-snhqh:/# intkey list
WzmnpK: 83082
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
WzmnpK: 83082
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
WzmnpK: 83082
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
WzmnpK: 83082
pineapples: 5
root@pbft-0-6665c5795b-snhqh:/# intkey list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
WzmnpK: 83082
pineapples: 5
```

Checklist:

* Are there 2 intkey transactions on every node

```
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
2    0d146ca7d99a3682d92f31376b5d7452b644b207e37734a8611f6d96617baabd6505a5889b8c37e456a6403c0ecb348f858c90cfb1339c0437d408160331b5ef  2     9     02554e...
1    003bfc1e231187f68e674e9752360b9bec0a11260f83a14a32999786da2e46f042b44cca7887c730e8de696240a69ba9bdb12fdc5d84548e998c7ee061a025d6  1     1     02554e...
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
2    0d146ca7d99a3682d92f31376b5d7452b644b207e37734a8611f6d96617baabd6505a5889b8c37e456a6403c0ecb348f858c90cfb1339c0437d408160331b5ef  2     9     02554e...
1    003bfc1e231187f68e674e9752360b9bec0a11260f83a14a32999786da2e46f042b44cca7887c730e8de696240a69ba9bdb12fdc5d84548e998c7ee061a025d6  1     1     02554e...
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
2    0d146ca7d99a3682d92f31376b5d7452b644b207e37734a8611f6d96617baabd6505a5889b8c37e456a6403c0ecb348f858c90cfb1339c0437d408160331b5ef  2     9     02554e...
1    003bfc1e231187f68e674e9752360b9bec0a11260f83a14a32999786da2e46f042b44cca7887c730e8de696240a69ba9bdb12fdc5d84548e998c7ee061a025d6  1     1     02554e...
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
2    0d146ca7d99a3682d92f31376b5d7452b644b207e37734a8611f6d96617baabd6505a5889b8c37e456a6403c0ecb348f858c90cfb1339c0437d408160331b5ef  2     9     02554e...
1    003bfc1e231187f68e674e9752360b9bec0a11260f83a14a32999786da2e46f042b44cca7887c730e8de696240a69ba9bdb12fdc5d84548e998c7ee061a025d6  1     1     02554e...
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
root@pbft-0-6665c5795b-snhqh:/# sawtooth block list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER
2    0d146ca7d99a3682d92f31376b5d7452b644b207e37734a8611f6d96617baabd6505a5889b8c37e456a6403c0ecb348f858c90cfb1339c0437d408160331b5ef  2     9     02554e...
1    003bfc1e231187f68e674e9752360b9bec0a11260f83a14a32999786da2e46f042b44cca7887c730e8de696240a69ba9bdb12fdc5d84548e998c7ee061a025d6  1     1     02554e...
0    f809d2ef80f025c9335e25f1be05f52c7805002be757230a30c0532158db2f5b4b73dc3f627a1875ee0c452164f4a46493e86a37eeab69e99f107f08758ab7e9  2     5     036c14...
```

Checklist:

* Are there 3 blocks on the chain for every node

```
root@pbft-0-6665c5795b-snhqh:/# sawtooth peer list
tcp://10.233.43.72:8800,tcp://10.233.47.205:8800,tcp://10.233.48.178:8800,tcp://10.233.7.216:8800
root@pbft-0-6665c5795b-snhqh:/# sawtooth peer list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
tcp://10.233.43.72:8800,tcp://10.233.48.178:8800,tcp://10.233.48.178:8800,tcp://10.233.58.8:8800,tcp://10.233.7.216:8800
root@pbft-0-6665c5795b-snhqh:/# sawtooth peer list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
tcp://10.233.43.72:8800,tcp://10.233.47.205:8800,tcp://10.233.58.8:8800,tcp://10.233.7.216:8800
root@pbft-0-6665c5795b-snhqh:/# sawtooth peer list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
tcp://10.233.47.205:8800,tcp://10.233.48.178:8800,tcp://10.233.58.8:8800,tcp://10.233.7.216:8800
root@pbft-0-6665c5795b-snhqh:/# sawtooth peer list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
tcp://10.233.43.72:8800,tcp://10.233.47.205:8800,tcp://10.233.48.178:8800,tcp://10.233.58.8:8800
root@pbft-0-6665c5795b-snhqh:/# exit
```

Checklist:

* Is every node peered with every other node?

Now delete the Sawtooth network, configmaps, and related pods.

```
$ kubectl delete -f docker/kubernetes/sawtooth-kubernetes-default-pbft.yaml
```

Checklist: Check for this output after deleting sawtooth-kubernetes-default-pbft

* deployment.extensions "pbft-0" deleted
* service "sawtooth-0" deleted
* deployment.extensions "pbft-1" deleted
* service "sawtooth-1" deleted
* deployment.extensions "pbft-2" deleted
* service "sawtooth-2" deleted
* deployment.extensions "pbft-3" deleted
* service "sawtooth-3" deleted
* deployment.extensions "pbft-4" deleted
* service "sawtooth-4" deleted

```
$ kubectl delete -f docker/kubernetes/pbft-keys-configmap.yaml
```

Checklist: Has keys-config been deleted

* configmap "keys-config" deleted

```
$ kubectl delete -f docker/kubernetes/sawtooth-create-pbft-keys.yaml
```

Checklist:

* Has pbft-keys been deleted.

### Kubernetes - POET Consensus ###

This yaml file will create a five pod Sawtooth network running the POET
consensus engine on an Kubernetes cluster using the published Dockerhub images.

```
$ kubectl apply -f docker/kubernetes/sawtooth-kubernetes-default-poet.yaml 
deployment.extensions/sawtooth-0 created
service/sawtooth-0 created
deployment.extensions/sawtooth-1 created
service/sawtooth-1 created
deployment.extensions/sawtooth-2 created
service/sawtooth-2 created
deployment.extensions/sawtooth-3 created
service/sawtooth-3 created
deployment.extensions/sawtooth-4 created
service/sawtooth-4 created
```

Get the pod name.

```
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
sawtooth-0-7695b87956-qknm5   8/8     Running   0          8s
sawtooth-1-7dc674cf65-lf25z   8/8     Running   0          8s
sawtooth-2-6bd66bf4f7-mt2j7   8/8     Running   0          8s
sawtooth-3-5d7b65cfd5-f2n8c   8/8     Running   0          8s
sawtooth-4-d9574fd69-7pszc    8/8     Running   0          8s
```

Checklist:

* are 8 of 8 Containers ready in all five pods
* Do the 5 running pods have 0 restarts

Enter the shell container in the pod.

```
$ kubectl exec -it sawtooth-0-7695b87956-qknm5 --container sawtooth-shell -- bash
root@sawtooth-0-7695b87956-qknm5:/#
```

Run a few tests inside this container.

First we will show the block list, observing we have the Genesis block.

```
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
```

Checklist:

* Are blocks 0 (genesis) and 1 present

Next we will use intkey to set a value.

```
root@sawtooth-0-7695b87956-qknm5:/# intkey set pineapples 5
{
  "link": "http://127.0.0.1:8008/batch_statuses?id=da68a04af0b16122510300a3b978f1d8f2dbf98a6fd83df20219e39c77a66e3232131e3f0f76cd3124db0e176fd495900fa098cdd0db7a247358f8f4127ec943"
}
```

Checklist:

* Did this set command exit without error

Generate a batch with increment and decrement transactions then load it.

```
root@sawtooth-0-7695b87956-qknm5:/# intkey create_batch
Writing to batches.intkey...
root@sawtooth-0-7695b87956-qknm5:/# intkey load        
batches: 2 batch/sec: 136.28046918153166
```

## Verification Step ###

Finally check the intkey transactions we loaded, and list the blocks in the
chain to show blocks have been committed to the chain.

```
root@sawtooth-0-7695b87956-qknm5:/# intkey show pineapples
pineapples: 5
root@sawtooth-0-7695b87956-qknm5:/# intkey show pineapples --url http://$SAWTOOTH_1_SERVICE_HOST:8008
pineapples: 5
root@sawtooth-0-7695b87956-qknm5:/# intkey show pineapples --url http://$SAWTOOTH_2_SERVICE_HOST:8008
pineapples: 5
root@sawtooth-0-7695b87956-qknm5:/# intkey show pineapples --url http://$SAWTOOTH_3_SERVICE_HOST:8008
pineapples: 5
root@sawtooth-0-7695b87956-qknm5:/# intkey show pineapples --url http://$SAWTOOTH_4_SERVICE_HOST:8008
pineapples: 5
```

Checklist:

* Is pineapples equal to 5 on every node

```
root@sawtooth-0-7695b87956-qknm5:/# intkey list
pineapples: 5
QAzAtM: 72548
root@sawtooth-0-7695b87956-qknm5:/# intkey list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
pineapples: 5
QAzAtM: 72548
root@sawtooth-0-7695b87956-qknm5:/# intkey list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
pineapples: 5
QAzAtM: 72548
root@sawtooth-0-7695b87956-qknm5:/# intkey list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
pineapples: 5
QAzAtM: 72548
root@sawtooth-0-7695b87956-qknm5:/# intkey list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
pineapples: 5
QAzAtM: 72548
```

Checklist:

* Are there 2 intkey transactions on every node

```
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
3    f55e7bbeb01009b5c50efa7cc9d9ca0b9a50244a4feb6e8797f65e04656643eb4e89a0a0f223ad393b16adb2411586ff3f87a7aaa5a3dd2d67f6c24f5c36cc93  2     4     038824c0...
2    b5fbc4889ab83a7c867c11305caf566ad8d53df5165c80ac0a2bf2076aa805145e4c05304bd63f933da8f5e7135e1a80274e9f366e2d2ab3ccee5bd29fe7ce8a  1     1     03856812...
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
3    f55e7bbeb01009b5c50efa7cc9d9ca0b9a50244a4feb6e8797f65e04656643eb4e89a0a0f223ad393b16adb2411586ff3f87a7aaa5a3dd2d67f6c24f5c36cc93  2     4     038824c0...
2    b5fbc4889ab83a7c867c11305caf566ad8d53df5165c80ac0a2bf2076aa805145e4c05304bd63f933da8f5e7135e1a80274e9f366e2d2ab3ccee5bd29fe7ce8a  1     1     03856812...
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
3    f55e7bbeb01009b5c50efa7cc9d9ca0b9a50244a4feb6e8797f65e04656643eb4e89a0a0f223ad393b16adb2411586ff3f87a7aaa5a3dd2d67f6c24f5c36cc93  2     4     038824c0...
2    b5fbc4889ab83a7c867c11305caf566ad8d53df5165c80ac0a2bf2076aa805145e4c05304bd63f933da8f5e7135e1a80274e9f366e2d2ab3ccee5bd29fe7ce8a  1     1     03856812...
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
3    f55e7bbeb01009b5c50efa7cc9d9ca0b9a50244a4feb6e8797f65e04656643eb4e89a0a0f223ad393b16adb2411586ff3f87a7aaa5a3dd2d67f6c24f5c36cc93  2     4     038824c0...
2    b5fbc4889ab83a7c867c11305caf566ad8d53df5165c80ac0a2bf2076aa805145e4c05304bd63f933da8f5e7135e1a80274e9f366e2d2ab3ccee5bd29fe7ce8a  1     1     03856812...
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
root@sawtooth-0-7695b87956-qknm5:/# sawtooth block list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
NUM  BLOCK_ID                                                                                                                          BATS  TXNS  SIGNER  
3    f55e7bbeb01009b5c50efa7cc9d9ca0b9a50244a4feb6e8797f65e04656643eb4e89a0a0f223ad393b16adb2411586ff3f87a7aaa5a3dd2d67f6c24f5c36cc93  2     4     038824c0...
2    b5fbc4889ab83a7c867c11305caf566ad8d53df5165c80ac0a2bf2076aa805145e4c05304bd63f933da8f5e7135e1a80274e9f366e2d2ab3ccee5bd29fe7ce8a  1     1     03856812...
1    506bf7905c08ac4e132a73865bda10e9c5be440dc97cfd88605ac522430b4ae37e7b16217220e720e6e6bdedbb2df92127eac6d4bca7cd0ccb6d7c484cfedb4a  4     4     03856812...
0    3fe7b760ca20fdf2e21427c470343c1205e593a8decaa686eb6c7d076f66e67c49688b37894780cb31ff883af711557fbfa4235b0a6d73e65b8fbf9f58c6160d  3     12    03856812...
```

Checklist:

* Are there 4 blocks on the chain for every node

```
root@sawtooth-0-7695b87956-qknm5:/# sawtooth peer list
tcp://10.233.12.219:8800,tcp://10.233.22.250:8800,tcp://10.233.22.250:8800,tcp://10.233.27.229:8800,tcp://10.233.27.229:8800,tcp://10.233.33.130:8800,tcp://10.233.33.130:8800
root@sawtooth-0-7695b87956-qknm5:/# sawtooth peer list --url http://$SAWTOOTH_1_SERVICE_HOST:8008
tcp://10.233.12.219:8800,tcp://10.233.12.219:8800,tcp://10.233.22.250:8800,tcp://10.233.22.250:8800,tcp://10.233.27.229:8800,tcp://10.233.27.229:8800,tcp://10.233.8.212:8800,tcp://10.233.8.212:8800
root@sawtooth-0-7695b87956-qknm5:/# sawtooth peer list --url http://$SAWTOOTH_2_SERVICE_HOST:8008
tcp://10.233.12.219:8800,tcp://10.233.12.219:8800,tcp://10.233.22.250:8800,tcp://10.233.22.250:8800,tcp://10.233.33.130:8800,tcp://10.233.33.130:8800,tcp://10.233.8.212:8800,tcp://10.233.8.212:8800
root@sawtooth-0-7695b87956-qknm5:/# sawtooth peer list --url http://$SAWTOOTH_3_SERVICE_HOST:8008
tcp://10.233.12.219:8800,tcp://10.233.12.219:8800,tcp://10.233.27.229:8800,tcp://10.233.27.229:8800,tcp://10.233.33.130:8800,tcp://10.233.33.130:8800,tcp://10.233.8.212:8800,tcp://10.233.8.212:8800
root@sawtooth-0-7695b87956-qknm5:/# sawtooth peer list --url http://$SAWTOOTH_4_SERVICE_HOST:8008
tcp://10.233.22.250:8800,tcp://10.233.22.250:8800,tcp://10.233.27.229:8800,tcp://10.233.27.229:8800,tcp://10.233.33.130:8800,tcp://10.233.33.130:8800,tcp://10.233.8.212:8800
root@sawtooth-0-7695b87956-qknm5:/# exit
```

Checklist:

* Is every node peered with every other node?

Now delete the Sawtooth network

```
$ kubectl delete -f docker/kubernetes/sawtooth-kubernetes-default-poet.yaml 
deployment.extensions "sawtooth-0" deleted
```

Checklist: Check for this output after deleting sawtooth-kubernetes-default-poet

* service "sawtooth-0" deleted
* deployment.extensions "sawtooth-1" deleted
* service "sawtooth-1" deleted
* deployment.extensions "sawtooth-2" deleted
* service "sawtooth-2" deleted
* deployment.extensions "sawtooth-3" deleted
* service "sawtooth-3" deleted
* deployment.extensions "sawtooth-4" deleted
* service "sawtooth-4" deleted

### Final ###

We have now run sawtooth from the published Docker Images inside Kubernetes
using the provided demo files. 
