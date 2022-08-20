## Distributed Services with Go: Commit Log Distributed App/Service from Travis Jeffery's book

This repository follows the book "Distributed Services with Go" by Travis Jeffrey with some fixes (https://github.com/varunbpatil/proglog/commit/2ecd0d7812b40583b76594eb148c1b4e60ca835b && https://github.com/evdzhurov/dist-services-with-go/issues/1 && etc.)

The official repository for this book can be found at https://github.com/travisjeffery/proglog


This repository is a example of Disributed App with Go (gRPC/Service Discovery:Serf/Consensus:Raft/etc.)

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
## Install cfssl & cfssljson, make tests, build docker image 

```
### This will download, build, and install all of the utility programs (including cfssl, cfssljson, and mkbundle among others).
$ go install github.com/cloudflare/cfssl/cmd/...@latest


$ make init
$ make gencert
$ make test

### Build docker image:
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
2022-08-20T14:16:20.234Z [INFO]  raft: initial configuration: index=1 servers="[{Suffrage:Voter ID:proglog-0 Address:proglog-0.proglog.default.svc.cluster.local:8400}]"
2022-08-20T14:16:20.235Z [INFO]  raft: entering follower state: follower="Node at [::]:8400 [Follower]" leader=
2022-08-20T14:16:22.088Z [WARN]  raft: heartbeat timeout reached, starting election: last-leader=
2022-08-20T14:16:22.088Z [INFO]  raft: entering candidate state: node="Node at [::]:8400 [Candidate]" term=452
2022-08-20T14:16:22.103Z [DEBUG] raft: votes: needed=1
2022-08-20T14:16:22.103Z [DEBUG] raft: vote granted: from=proglog-0 term=452 tally=1
2022-08-20T14:16:22.103Z [INFO]  raft: election won: tally=1
2022-08-20T14:16:22.103Z [INFO]  raft: entering leader state: leader="Node at [::]:8400 [Leader]"
2022/08/20 14:16:22 [INFO] serf: EventMemberJoin: proglog-0 192.168.188.180
2022-08-20T14:16:34.957Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:34Z", "grpc.request.deadline": "2022-08-20T14:16:35Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:57948", "grpc.code": "OK", "grpc.time_ns": 87388}
2022-08-20T14:16:34.958Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:34Z", "grpc.request.deadline": "2022-08-20T14:16:35Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:57950", "grpc.code": "OK", "grpc.time_ns": 619134}
2022/08/20 14:16:37 [DEBUG] memberlist: Stream connection from=192.168.188.181:52396
2022/08/20 14:16:37 [INFO] serf: EventMemberJoin: proglog-1 192.168.188.181
2022-08-20T14:16:37.372Z [INFO]  raft: updating configuration: command=AddStaging server-id=proglog-1 server-addr=proglog-1.proglog.default.svc.cluster.local:8400 servers="[{Suffrage:Voter ID:proglog-0 Address:proglog-0.proglog.default.svc.cluster.local:8400} {Suffrage:Voter ID:proglog-1 Address:proglog-1.proglog.default.svc.cluster.local:8400}]"
2022-08-20T14:16:37.372Z [INFO]  raft: added peer, starting replication: peer=proglog-1
2022-08-20T14:16:37.379Z [ERROR] raft: failed to appendEntries to: peer="{Voter proglog-1 proglog-1.proglog.default.svc.cluster.local:8400}" error="dial tcp: lookup proglog-1.proglog.default.svc.cluster.local on 10.96.0.10:53: no such host"
2022-08-20T14:16:37.497Z [ERROR] raft: failed to heartbeat to: peer=proglog-1.proglog.default.svc.cluster.local:8400 error="dial tcp: lookup proglog-1.proglog.default.svc.cluster.local on 10.96.0.10:53: no such host"
2022/08/20 14:16:37 [DEBUG] serf: messageJoinType: proglog-1
2022-08-20T14:16:37.637Z [ERROR] raft: failed to heartbeat to: peer=proglog-1.proglog.default.svc.cluster.local:8400 error="dial tcp: lookup proglog-1.proglog.default.svc.cluster.local on 10.96.0.10:53: no such host"
2022/08/20 14:16:37 [DEBUG] serf: messageJoinType: proglog-1
2022-08-20T14:16:37.793Z [ERROR] raft: failed to heartbeat to: peer=proglog-1.proglog.default.svc.cluster.local:8400 error="dial tcp: lookup proglog-1.proglog.default.svc.cluster.local on 10.96.0.10:53: no such host"
2022-08-20T14:16:37.873Z [WARN]  raft: failed to contact: server-id=proglog-1 time=500.209767ms
2022-08-20T14:16:37.873Z [WARN]  raft: failed to contact quorum of nodes, stepping down
2022-08-20T14:16:37.873Z [INFO]  raft: entering follower state: follower="Node at [::]:8400 [Follower]" leader=
2022/08/20 14:16:37 [DEBUG] serf: messageJoinType: proglog-1
2022/08/20 14:16:38 [DEBUG] serf: messageJoinType: proglog-1
2022-08-20T14:16:39.579Z [WARN]  raft: heartbeat timeout reached, starting election: last-leader=
2022-08-20T14:16:39.580Z [INFO]  raft: entering candidate state: node="Node at [::]:8400 [Candidate]" term=453
2022-08-20T14:16:39.647Z [DEBUG] raft: votes: needed=2
2022-08-20T14:16:39.647Z [DEBUG] raft: vote granted: from=proglog-0 term=453 tally=1
2022-08-20T14:16:39.731Z [DEBUG] raft: vote granted: from=proglog-1 term=453 tally=2
2022-08-20T14:16:39.731Z [INFO]  raft: election won: tally=2
2022-08-20T14:16:39.731Z [INFO]  raft: entering leader state: leader="Node at [::]:8400 [Leader]"
2022-08-20T14:16:39.731Z [INFO]  raft: added peer, starting replication: peer=proglog-1
2022-08-20T14:16:39.734Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-1 proglog-1.proglog.default.svc.cluster.local:8400}" next=2
2022-08-20T14:16:39.745Z [INFO]  raft: pipelining replication: peer="{Voter proglog-1 proglog-1.proglog.default.svc.cluster.local:8400}"
2022-08-20T14:16:39.902Z [ERROR] raft: failed to heartbeat to: peer=proglog-1.proglog.default.svc.cluster.local:8400 error="dial tcp: lookup proglog-1.proglog.default.svc.cluster.local on 10.96.0.10:53: no such host"
2022-08-20T14:16:44.937Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:44Z", "grpc.request.deadline": "2022-08-20T14:16:45Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58178", "grpc.code": "OK", "grpc.time_ns": 32102}
2022-08-20T14:16:44.949Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:44Z", "grpc.request.deadline": "2022-08-20T14:16:45Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58180", "grpc.code": "OK", "grpc.time_ns": 44949}
2022-08-20T14:16:54.930Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:54Z", "grpc.request.deadline": "2022-08-20T14:16:55Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58382", "grpc.code": "OK", "grpc.time_ns": 51235}
2022-08-20T14:16:54.944Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:54Z", "grpc.request.deadline": "2022-08-20T14:16:55Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58384", "grpc.code": "OK", "grpc.time_ns": 37445}
2022/08/20 14:16:56 [DEBUG] memberlist: Stream connection from=192.168.188.178:60402
2022/08/20 14:16:56 [INFO] serf: EventMemberJoin: proglog-2 192.168.188.178
2022-08-20T14:16:56.506Z [INFO]  raft: updating configuration: command=AddStaging server-id=proglog-2 server-addr=proglog-2.proglog.default.svc.cluster.local:8400 servers="[{Suffrage:Voter ID:proglog-0 Address:proglog-0.proglog.default.svc.cluster.local:8400} {Suffrage:Voter ID:proglog-1 Address:proglog-1.proglog.default.svc.cluster.local:8400} {Suffrage:Voter ID:proglog-2 Address:proglog-2.proglog.default.svc.cluster.local:8400}]"
2022-08-20T14:16:56.506Z [INFO]  raft: added peer, starting replication: peer=proglog-2
2022-08-20T14:16:56.510Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" next=9
2022-08-20T14:16:56.511Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" next=8
2022-08-20T14:16:56.511Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" next=7
2022-08-20T14:16:56.511Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" next=6
2022-08-20T14:16:56.522Z [ERROR] raft: failed to appendEntries to: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" error=EOF
2022-08-20T14:16:56.668Z [ERROR] raft: failed to heartbeat to: peer=proglog-2.proglog.default.svc.cluster.local:8400 error="dial tcp 192.168.188.178:8400: connect: connection refused"
2022-08-20T14:16:56.820Z [ERROR] raft: failed to heartbeat to: peer=proglog-2.proglog.default.svc.cluster.local:8400 error="dial tcp 192.168.188.178:8400: connect: connection refused"
2022-08-20T14:16:57.011Z [WARN]  raft: failed to contact: server-id=proglog-2 time=500.216823ms
2022-08-20T14:16:57.018Z [ERROR] raft: failed to heartbeat to: peer=proglog-2.proglog.default.svc.cluster.local:8400 error="dial tcp 192.168.188.178:8400: connect: connection refused"
2022-08-20T14:16:57.225Z [ERROR] raft: failed to heartbeat to: peer=proglog-2.proglog.default.svc.cluster.local:8400 error="dial tcp 192.168.188.178:8400: connect: connection refused"
2022-08-20T14:16:57.432Z [WARN]  raft: failed to contact: server-id=proglog-2 time=920.434585ms
2022-08-20T14:16:57.473Z [ERROR] raft: failed to heartbeat to: peer=proglog-2.proglog.default.svc.cluster.local:8400 error="dial tcp 192.168.188.178:8400: connect: connection refused"
2022/08/20 14:16:57 [DEBUG] memberlist: Stream connection from=192.168.188.178:60432
2022/08/20 14:16:57 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:57 [DEBUG] serf: messageJoinType: proglog-2
2022-08-20T14:16:58.001Z [WARN]  raft: appendEntries rejected, sending older logs: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}" next=1
2022-08-20T14:16:58.002Z [INFO]  raft: pipelining replication: peer="{Voter proglog-2 proglog-2.proglog.default.svc.cluster.local:8400}"
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:17:01 [DEBUG] memberlist: Initiating push/pull sync with: proglog-1 192.168.188.181:8401
2022-08-20T14:17:04.933Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:04Z", "grpc.request.deadline": "2022-08-20T14:17:05Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58600", "grpc.code": "OK", "grpc.time_ns": 42278}
2022-08-20T14:17:04.944Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:04Z", "grpc.request.deadline": "2022-08-20T14:17:05Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58602", "grpc.code": "OK", "grpc.time_ns": 35585}
2022-08-20T14:17:14.923Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:14Z", "grpc.request.deadline": "2022-08-20T14:17:15Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58790", "grpc.code": "OK", "grpc.time_ns": 42928}
2022-08-20T14:17:14.937Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:14Z", "grpc.request.deadline": "2022-08-20T14:17:15Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58792", "grpc.code": "OK", "grpc.time_ns": 48476}
2022/08/20 14:17:16 [DEBUG] memberlist: Stream connection from=192.168.188.181:53238
2022-08-20T14:17:24.929Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:24Z", "grpc.request.deadline": "2022-08-20T14:17:25Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:58998", "grpc.code": "OK", "grpc.time_ns": 38043}
2022-08-20T14:17:24.941Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:24Z", "grpc.request.deadline": "2022-08-20T14:17:25Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:59000", "grpc.code": "OK", "grpc.time_ns": 31262}
2022/08/20 14:17:29 [DEBUG] memberlist: Stream connection from=192.168.188.178:32890
2022/08/20 14:17:31 [DEBUG] memberlist: Initiating push/pull sync with: proglog-1 192.168.188.181:8401
2022-08-20T14:17:34.937Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:34Z", "grpc.request.deadline": "2022-08-20T14:17:35Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:59192", "grpc.code": "OK", "grpc.time_ns": 615296}
2022-08-20T14:17:34.944Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:34Z", "grpc.request.deadline": "2022-08-20T14:17:35Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.180:59194", "grpc.code": "OK", "grpc.time_ns": 34727}

$ kubectl logs proglog-1
2022-08-20T14:16:37.365Z [INFO]  raft: initial configuration: index=1 servers="[{Suffrage:Voter ID:proglog-0 Address:proglog-0.proglog.default.svc.cluster.local:8400}]"
2022-08-20T14:16:37.366Z [INFO]  raft: entering follower state: follower="Node at [::]:8400 [Follower]" leader=
2022/08/20 14:16:37 [INFO] serf: EventMemberJoin: proglog-1 192.168.188.181
2022/08/20 14:16:37 [DEBUG] memberlist: Initiating push/pull sync with:  192.168.188.180:8401
2022/08/20 14:16:37 [INFO] serf: EventMemberJoin: proglog-0 192.168.188.180
2022/08/20 14:16:37 [DEBUG] serf: messageJoinType: proglog-1
2022/08/20 14:16:37 [DEBUG] serf: messageJoinType: proglog-1
2022/08/20 14:16:38 [DEBUG] serf: messageJoinType: proglog-1
2022/08/20 14:16:38 [DEBUG] serf: messageJoinType: proglog-1
2022-08-20T14:16:38.983Z [WARN]  raft: heartbeat timeout reached, not part of stable configuration, not triggering a leader election
2022-08-20T14:16:39.662Z [DEBUG] raft: lost leadership because received a requestVote with a newer term
2022-08-20T14:16:39.732Z [WARN]  raft: failed to get previous log: previous-index=7 last-index=1 error="rpc error: code = NotFound desc = offset out of range: 7"
2022-08-20T14:16:55.049Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:55Z", "grpc.request.deadline": "2022-08-20T14:16:56Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:57736", "grpc.code": "OK", "grpc.time_ns": 27885}
2022-08-20T14:16:55.062Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:16:55Z", "grpc.request.deadline": "2022-08-20T14:16:56Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:57738", "grpc.code": "OK", "grpc.time_ns": 42593}
2022/08/20 14:16:56 [INFO] serf: EventMemberJoin: proglog-2 192.168.188.178
2022/08/20 14:16:57 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:17:01 [DEBUG] memberlist: Stream connection from=192.168.188.180:49166
2022-08-20T14:17:05.048Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:05Z", "grpc.request.deadline": "2022-08-20T14:17:06Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:57954", "grpc.code": "OK", "grpc.time_ns": 93235}
2022-08-20T14:17:05.060Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:05Z", "grpc.request.deadline": "2022-08-20T14:17:06Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:57956", "grpc.code": "OK", "grpc.time_ns": 35289}
2022-08-20T14:17:15.048Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:15Z", "grpc.request.deadline": "2022-08-20T14:17:16Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58144", "grpc.code": "OK", "grpc.time_ns": 42354}
2022-08-20T14:17:15.060Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:15Z", "grpc.request.deadline": "2022-08-20T14:17:16Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58146", "grpc.code": "OK", "grpc.time_ns": 40890}
2022/08/20 14:17:16 [DEBUG] memberlist: Initiating push/pull sync with: proglog-0 192.168.188.180:8401
2022-08-20T14:17:25.050Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:25Z", "grpc.request.deadline": "2022-08-20T14:17:26Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58352", "grpc.code": "OK", "grpc.time_ns": 40765}
2022-08-20T14:17:25.063Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:25Z", "grpc.request.deadline": "2022-08-20T14:17:26Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58354", "grpc.code": "OK", "grpc.time_ns": 29705}
2022/08/20 14:17:31 [DEBUG] memberlist: Stream connection from=192.168.188.180:49758
2022-08-20T14:17:35.049Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:35Z", "grpc.request.deadline": "2022-08-20T14:17:36Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58546", "grpc.code": "OK", "grpc.time_ns": 45161}
2022-08-20T14:17:35.061Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:35Z", "grpc.request.deadline": "2022-08-20T14:17:36Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.181:58548", "grpc.code": "OK", "grpc.time_ns": 34606}

$ kubectl logs proglog-2
2022-08-20T14:16:57.627Z [INFO]  raft: initial configuration: index=0 servers=[]
2022-08-20T14:16:57.627Z [INFO]  raft: entering follower state: follower="Node at [::]:8400 [Follower]" leader=
2022/08/20 14:16:57 [INFO] serf: EventMemberJoin: proglog-2 192.168.188.178
2022/08/20 14:16:57 [DEBUG] memberlist: Initiating push/pull sync with:  192.168.188.180:8401
2022/08/20 14:16:57 [INFO] serf: EventMemberJoin: proglog-0 192.168.188.180
2022/08/20 14:16:57 [INFO] serf: EventMemberJoin: proglog-1 192.168.188.181
2022-08-20T14:16:57.630Z	DEBUG	membership	discovery/membership.go:125	failed to join	{"error": "node is not the leader", "name": "proglog-0", "rpc_addr": "proglog-0.proglog.default.svc.cluster.local:8400"}
2022-08-20T14:16:57.630Z	DEBUG	membership	discovery/membership.go:125	failed to join	{"error": "node is not the leader", "name": "proglog-1", "rpc_addr": "proglog-1.proglog.default.svc.cluster.local:8400"}
2022/08/20 14:16:57 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:57 [DEBUG] serf: messageJoinType: proglog-2
2022-08-20T14:16:58.000Z [WARN]  raft: failed to get previous log: previous-index=5 last-index=0 error="rpc error: code = NotFound desc = offset out of range: 5"
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022/08/20 14:16:58 [DEBUG] serf: messageJoinType: proglog-2
2022-08-20T14:17:15.135Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:15Z", "grpc.request.deadline": "2022-08-20T14:17:16Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:48682", "grpc.code": "OK", "grpc.time_ns": 108808}
2022-08-20T14:17:15.147Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:15Z", "grpc.request.deadline": "2022-08-20T14:17:16Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:48684", "grpc.code": "OK", "grpc.time_ns": 45136}
2022-08-20T14:17:25.136Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:25Z", "grpc.request.deadline": "2022-08-20T14:17:26Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:48890", "grpc.code": "OK", "grpc.time_ns": 46963}
2022-08-20T14:17:25.148Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:25Z", "grpc.request.deadline": "2022-08-20T14:17:26Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:48892", "grpc.code": "OK", "grpc.time_ns": 48325}
2022/08/20 14:17:29 [DEBUG] memberlist: Initiating push/pull sync with: proglog-0 192.168.188.180:8401
2022-08-20T14:17:35.137Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:35Z", "grpc.request.deadline": "2022-08-20T14:17:36Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:49084", "grpc.code": "OK", "grpc.time_ns": 38037}
2022-08-20T14:17:35.149Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:35Z", "grpc.request.deadline": "2022-08-20T14:17:36Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:49086", "grpc.code": "OK", "grpc.time_ns": 26673}
2022-08-20T14:17:45.136Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:45Z", "grpc.request.deadline": "2022-08-20T14:17:46Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:49292", "grpc.code": "OK", "grpc.time_ns": 36762}
2022-08-20T14:17:45.150Z	INFO	server	zap/options.go:212	finished unary call with code OK	{"grpc.start_time": "2022-08-20T14:17:45Z", "grpc.request.deadline": "2022-08-20T14:17:46Z", "system": "grpc", "span.kind": "server", "grpc.service": "grpc.health.v1.Health", "grpc.method": "Check", "peer.address": "192.168.188.178:49294", "grpc.code": "OK", "grpc.time_ns": 42081}

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
