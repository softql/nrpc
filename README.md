# nRPC

[![Build Status](https://travis-ci.org/rapidloop/nrpc.svg?branch=master)](https://travis-ci.org/rapidloop/nrpc)

nRPC is an RPC framework like [gRPC]((https://grpc.io/), but for
[NATS](https://nats.io/).

It can generate a Go client and server from the same .proto file that you'd
use to generate gRPC clients and servers. The server is generated as a NATS
[MsgHandler](https://godoc.org/github.com/nats-io/go-nats#MsgHandler).

## Why NATS?

Doing RPC over NATS'
[request-response model](http://nats.io/documentation/concepts/nats-req-rep/)
has some advantages over a gRPC model:

- **Minimal service discovery**: The clients and servers only need to know the
  endpoints of a NATS cluster. The clients do not need to discover the
  endpoints of individual services they depend on.
- **Load balancing without load balancers**: Stateless microservices can be
  hosted redundantly and connected to the same NATS cluster. The incoming
  requests can then be random-routed among these using NATS
  [queueing](http://nats.io/documentation/concepts/nats-queueing/). There is
  no need to setup a (high availability) load balancer per microservice.

The lunch is not always free, however. At scale, the NATS cluster itself can
become a bottleneck. Features of gRPC like streaming and advanced auth are not
available.

Still, NATS - and nRPC - offer much lower operational complexity if your
scale and requirements fit.

At RapidLoop, we use this model for our [OpsDash](https://www.opsdash.com)
SaaS product in production and are quite happy with it. nRPC is the third
iteration of an internal library.

## Overview

nRPC comes with a protobuf compiler plugin `protoc-gen-nrpc`, which generates
Go code from a .proto file.

Given a .proto file like [helloworld.proto](https://github.com/grpc/grpc-go/blob/master/examples/helloworld/helloworld/helloworld.proto), the usage is like this:

```
$ ls
helloworld.proto
$ protoc --go_out=. --nrpc_out=. helloworld.proto
$ ls
helloworld.nrpc.go	helloworld.pb.go	helloworld.proto
```

The .pb.go file, which contains the definitions for the message classes, is
generated by the standard Go plugin for protoc. The .nrpc.go file, which
contains the definitions for a client, a server interface, and a NATS handler
is generated by the nRPC plugin.

Have a look at the generated and example files:

- the service definition [helloworld.proto](https://github.com/rapidloop/nrpc/tree/master/examples/helloworld/helloworld/helloworld.proto)
- the generated nrpc go file [helloworld.nrpc.go](https://github.com/rapidloop/nrpc/tree/master/examples/helloworld/helloworld/helloworld.nrpc.go)
- an example server [greeter_server/main.go](https://github.com/rapidloop/nrpc/tree/master/examples/helloworld/greeter_server/main.go)
- an example client [greeter_client/main.go](https://github.com/rapidloop/nrpc/tree/master/examples/helloworld/greeter_client/main.go)

## Installation

nRPC needs Go 1.7 or higher. $GOPATH/bin needs to be in $PATH for the protoc
invocation to work.

To install the nRPC protoc plugin:

```
$ go get github.com/rapidloop/nrpc/protoc-gen-nrpc
```

To build and run the example greeter_server:

```
$ go get github.com/rapidloop/nrpc/examples/greeter_server
$ greeter_server
server is running, ^C quits.
```

To build and run the example greeter_client:

```
$ go get github.com/rapidloop/nrpc/examples/greeter_client
$ greeter_client
Greeting: Hello world
$
```

## Documentation

To learn more about describing gRPC services using .proto files, see [here](https://grpc.io/docs/guides/concepts.html).
To learn more about NATS, start with their [website](https://nats.io/). To
learn more about nRPC, um, read the source code.

## Status

nRPC is in alpha. This means that it will work, but APIs may change without
notice.

Currently there is support only for Go clients and servers.

Built by RapidLoop. Released under Apache 2.0 license.

