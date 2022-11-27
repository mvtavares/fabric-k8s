# Kubernetes Fabric Network 

This project re-establishes the Hyperledger [liberty-dlt](../liberty-dlt) as a _cloud native_ application.

## Prerequisites:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [jq](https://stedolan.github.io/jq/)
- [envsubst](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html) (`brew install gettext` on OSX)

- K8s - either:
  - [KIND](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) + [Docker](https://www.docker.com) (resources: 8 CPU / 8 GRAM) 
  - [Rancher Desktop](https://rancherdesktop.io) (resources: 8 CPU / 8GRAM, mobyd, and disable Traefik)

## Quickstart 

Create a KIND cluster:  
```shell
./network kind
./network cluster init
```
or for [Rancher / k3s](docs/KUBERNETES.md#rancher-desktop-and-k3s):
```shell
export NETWORK_CLUSTER_RUNTIME=k3s

./network cluster init
```

Launch the network, create a channel, and deploy the [basic-asset-transfer](../asset-transfer-basic) smart contract: 
```shell
./network up

./network channel create

./network chaincode deploy asset-transfer-basic ../asset-transfer-basic/chaincode-java
```

Invoke and query chaincode:
```shell
./network chaincode invoke asset-transfer-basic '{"Args":["InitLedger"]}'
./network chaincode query  asset-transfer-basic '{"Args":["ReadAsset","asset1"]}'
```

Access the blockchain with a [REST API](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-basic/rest-api-typescript): 
```shell
./network rest-easy
```

Shut down the test network: 
```shell
./network down 
```

Tear down the cluster (KIND): 
```shell
./network unkind
```

For Rancher: Preferences -> Kubernetes Settings -> Reset Kubernetes  OR ...
```shell
./network cluster clean
```


## [Detailed Guides](docs/README.md)

- [Working with Kubernetes](docs/KUBERNETES.md)
- [Certificate Authorities](docs/CA.md)
- [Launching the Test Network](docs/NETWORK.md)
- [Working with Channels](docs/CHANNELS.md)
- [Working with Chaincode](docs/CHAINCODE.md)
- [Working with Applications](docs/APPLICATIONS.md)


### DNS Resolution on OSX

Fabric's OSX binaries have been statically linked with the golang `go` DNS resolver.  In some environments, this 
causes a brief but [noticeable delay](https://github.com/hyperledger/fabric/issues/3372) when issuing peer commands 
against the test network.

Workarounds to improve DNS resolution time on OSX: 

- Add manual DNS overrides for virtual hosts by adding to /etc/hosts:
```
127.0.0.1 org0-ca.localho.st
127.0.0.1 org1-ca.localho.st
127.0.0.1 org0-orderer1.localho.st
127.0.0.1 org0-orderer2.localho.st
127.0.0.1 org0-orderer3.localho.st
127.0.0.1 org1-peer1.localho.st
127.0.0.1 org1-peer2.localho.st
```

- Reduce the system resolver timeout from the default 5s by adding to /etc/resolv.conf:
```shell
options: timeout 2
```

- Compile the [fabric binaries](https://github.com/hyperledger/fabric) on a Mac and copy `build/bin/*` outputs to 
  `liberty-dlt-k8s/bin`.  Mac native builds are linked against the `netdns=cgo` DNS resolver, and are not
  subject to the timeouts associated with the Golang DNS resolver.
