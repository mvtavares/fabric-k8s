# Kubernetes Liberty Network 

Esse projeto foi adaptado a partir da test-network do Fabric-samples. 

* A validade dos certificados foi alterada de 1 ano para 5 anos.

## Pre requisitos:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [jq](https://stedolan.github.io/jq/)
- [envsubst](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html) (`brew install gettext` on OSX)

- K8s:
  - [KIND](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) + [Docker](https://www.docker.com) (resources: 8 CPU / 8 GRAM) 

## Quickstart 

Criar um KIND cluster:  
```shell
./network kind
```

Se já tiver um cluster instalado, apenas exportar o CLUSTER_RUNTIME conforme abaixo. Essa instalação fará tudo localmente através de port-forwards para acessar os pods necessários para inicialização da rede.
```shell
export NETWORK_CLUSTER_RUNTIME=k8s
```

Instalar Ingress Controller & Cert Manager
```shell
./network cluster init
```

Iniciar a rede, criar um canal e 

```shell
./network up

./network channel create
```

Para fazer o deploy do chaincode, é necessário publicar a imagem para algum registry e apontar a URL.
O valor default está apontando para ghcr.io/mvtavares mas pode ser alterado exportando o comando abaixo:

```shell
  export NETWORK_REMOTE_REGISTRY_NAME=ghcr.io/xxxx
```

Implantar o [basic-asset-transfer](../asset-transfer-basic) smart contract (chaincode):

```shell
./network chaincode deploy asset-transfer-basic ./chaincode
```

Initializar e consultar o chaincode:
```shell
./network chaincode invoke asset-transfer-basic '{"Args":["InitLedger"]}'
./network chaincode query asset-transfer-basic '{"Args":["ReadAsset","asset1"]}'
```

Terminar a rede: 
```shell
./network down 
```

Terminar o cluster (KIND): 
```shell
./network unkind
```

Limpar o cluster: 
```shell
./network cluster clean
```

Conectando com a aplicação:

ver exemplo: application-deployment.yaml

Configurar as variaveis para geração dos arquivos:

```shell
 export CHAINCODE_NAME=NOME_CHAINCODE
```

Exemplo: `export CHAINCODE_NAME=asset-transfer-basic`

```shell
 ./network application
```

Os arquivos de conexão serão gerados na pasta /build/application/gateways/ (Connection profile) e /build/application/wallet (identities para conexão com fabric).

## [Detailed Guides](docs/README.md)

- [Working with Kubernetes](docs/KUBERNETES.md)
- [Launching the Network](docs/NETWORK.md)
- [Working with Channels](docs/CHANNELS.md)
- [Working with Chaincode](docs/CHAINCODE.md)

Checando a expiração dos certificados:

```shell
kubectl get certificates -n liberty-dlt -o wide

NAME                     READY   SECRET                        ISSUER                 STATUS                                          AGE
org0-ca-tls-cert         True    org0-ca-tls-cert              org0-tls-cert-issuer   Certificate is up to date and has not expired   30m
org0-orderer1-tls-cert   True    org0-orderer1-tls-cert        org0-tls-cert-issuer   Certificate is up to date and has not expired   29m
org0-orderer2-tls-cert   True    org0-orderer2-tls-cert        org0-tls-cert-issuer   Certificate is up to date and has not expired   29m
org0-orderer3-tls-cert   True    org0-orderer3-tls-cert        org0-tls-cert-issuer   Certificate is up to date and has not expired   29m
org0-tls-cert-issuer     True    org0-tls-cert-issuer-secret   root-tls-cert-issuer   Certificate is up to date and has not expired   30m
org1-ca-tls-cert         True    org1-ca-tls-cert              org1-tls-cert-issuer   Certificate is up to date and has not expired   30m
org1-peer1-tls-cert      True    org1-peer1-tls-cert           org1-tls-cert-issuer   Certificate is up to date and has not expired   29m
org1-peer2-tls-cert      True    org1-peer2-tls-cert           org1-tls-cert-issuer   Certificate is up to date and has not expired   29m
org1-tls-cert-issuer     True    org1-tls-cert-issuer-secret   root-tls-cert-issuer   Certificate is up to date and has not expired   30m
```

Para exibir a data de início e fim dos certificados, é necessário descrevê-los conforme abaixo:

```shell
kubectl get certificate -n liberty-dlt -ojsonpath="{range .items[*]}{.metadata.name} not before: {.status.notBefore} not after: {.status.notAfter}{'\n'}{end}"
```
