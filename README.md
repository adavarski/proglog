## Distributed Services with Go: Commit Log from Travis Jeffery's book

This repository follows the book "Distributed Services with Go" by Travis Jeffrey with some fixes (https://github.com/varunbpatil/proglog/commit/2ecd0d7812b40583b76594eb148c1b4e60ca835b && https://github.com/evdzhurov/dist-services-with-go/issues/1)

The official repository for this book can be found at https://github.com/travisjeffery/proglog


This repository is a example of some gRPC capabilities in Go, using docker and k8s (for maximum scalability).

### Pre: Install Go 
```
$ wget https://go.dev/dl/go1.18.4.linux-amd64.tar.gz
$ sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz

$ grep GO ~/.bashrc 
export GOROOT=/usr/local/go
export PATH=${GOROOT}/bin:${PATH}
export GOPATH=$HOME/go
export PATH=${GOPATH}/bin:${PATH}

$ go version
go version go1.18.4 linux/amd64
```

### Compiling the proto files (gRPC):

Install protoc binary:
```
$ curl -LO $PB_REL/download/v3.15.8/protoc-3.15.8-linux-x86_64.zip
$ unzip protoc-3.15.8-linux-x86_64.zip -d $HOME/.local
$ export PATH="$PATH:$HOME/.local/bin"
$ protoc --version
libprotoc 3.15.8
```
Install the protocol compiler plugins for Go using the following commands (Ref: https://grpc.io/docs/languages/go/quickstart/):

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```
Fist we need to compile all files that will define our grpc protocol, all coded in the file functions/functions.proto with the following command:

```
$ cd api/v1/
$ protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative log.proto 
```
## Install cfssl & cfssljson 

```
### This will download, build, and install all of the utility programs (including cfssl, cfssljson, and mkbundle among others).

$ go install github.com/cloudflare/cfssl/cmd/...@latest
$ make init
$ make gencert
$ make test
etc.

```

### Build docker image:
```
$ make build-docker
$ docker push davarski/proglog:0.0.10
```
### K8s deploy
```

## Instal KIND

$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64 && chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

## Create cluster (CNI=Calico, Enable ingress)

$ kind create cluster --name devops --config cluster-config.yaml

$ kind get kubeconfig --name="devops" > admin.conf
$ export KUBECONFIG=./admin.conf 

$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
$ kubectl -n kube-system set env daemonset/calico-node FELIX_IGNORELOOSERPF=true

## Deploy Go distributed service:

$ helm install proglog deploy/proglog/
$ helm list

$ kubectl get po|grep prog
proglog-0                         1/1     Running   0               39m
proglog-1                         1/1     Running   1 (39m ago)     39m
proglog-2                         1/1     Running   1 (39m ago)     39m
$ kubectl logs proglog-0
$ kubectl logs proglog-1
$ kubectl logs proglog-2

$ kubectl get svc/proglog
NAME      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
proglog   ClusterIP   10.96.30.71   <none>        8400/TCP,8401/TCP,8401/UDP   39s

$ kubectl port-forward svc/proglog 8400
Forwarding from 127.0.0.1:8400 -> 8400
Forwarding from [::1]:8400 -> 8400
Handling connection for 8400

### Other terminal: 
$ go run cmd/getserver/main.go 
servers:
	- id:"proglog-0"  rpc_addr:"proglog-0.proglog.default.svc.cluster.local:8400"
	- id:"proglog-1"  rpc_addr:"proglog-1.proglog.default.svc.cluster.local:8400"
	- id:"proglog-2"  rpc_addr:"proglog-2.proglog.default.svc.cluster.local:8400"

```
