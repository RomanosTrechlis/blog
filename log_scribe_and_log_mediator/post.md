Recently, at work, I kept falling into a problem regarding log files. The logs are written on the machine the cluster node is in, making their retrieval time consuming. So I decided to tackle this problem with a custom solution that might never be introduced to production, however it will be an excellent weekend problem to solve.

So, what I want, is to make a system:

1. that writes logs to a single location for all cluster nodes and
1. at the same time respects the security policy we have.

The first thought was to implement an endpoint that receives something to log. What is necessary for logging is:

1. a relative path,
1. a filename (without the .log extension), and
1. a line to write.


## Communication

Looking around I found the gRPC framework for communication between client and server, and the protobuf language to write the structure of the messages exchanged between client and server.

This is the message sent to the server:
``` protobuf
message LogRequest {
  string filename = 1;
  string path = 2;
  string line = 3;
}
```

And I also defined an rpc service:
```protobuf
service LogScribe {
  // the LogScribe service sends a LogRequest
  // and recieves a LogResponse
  rpc Log(LogRequest) returns (LogResponse) {}
}
```

Btw, I named the server **logScribe**, because he scribes messages.

The only remaining thing is to compile the protobuf notation to something we can use.
For this we need the **protoc** command and a given language plugin, like go or java. So, in order to compile it to go code we use the command:
```shell
$ protoc --go_out=plugins=grpc:. *.proto
```
and in order to produce java code we use the command:
```shell
$ protoc --grpc-java_out=. \
  --plugin=protoc-gen-grpc=~/go/bin/protoc-gen-grpc-java \
  --java_out=. *.proto
```
where what goes after the protoc-gen-grpc is the path to the java plugin.

This combination gives an endpoint where all clusters can send log requests and the logScribe will write them on a single place.

## Security

So, in order to make it secure I tried something new for me. A two-way SSL authentication. This requires both the client and the server having SSL certificates, so both client and server trust each other.

I used certstrap in order to produce a dummy certificate authority, and then produce signed certificates for the server and the client:
```shell
$ certstrap --depot-path ./certs init --cn "CertAuth"

$ certstrap --depot-path certs request-cert -ip 127.0.0.1
$ certstrap --depot-path certs sign 127.0.0.1 --CA CertAuth
$ mv certs/127.0.0.1.crt certs/server.crt
$ mv certs/127.0.0.1.key certs/server.key
$ mv certs/127.0.0.1.csr certs/server.csr
$ certstrap --depot-path certs request-cert -ip 127.0.0.1
$ certstrap --depot-path certs sign 127.0.0.1 --CA CertAuth
$ mv certs/127.0.0.1.crt certs/client.crt
$ mv certs/127.0.0.1.key certs/client.key
$ mv certs/127.0.0.1.csr certs/client.csr
```

## Log Scribe Usage

Bellow are the flags for the logScribe.

```
Usage of logScribe:
  -ca string
    	certificate authority's certificate
  -console
    	dumps log lines to console
  -crt string
    	host's certificate for secured connections
  -mediator string
      mediators address if exists, i.e 127.0.0.1:8080
  -path string
    	path for logs to be persisted (default "../logs")
  -pk string
    	host's private key
  -port int
    	port for server to listen to requests (default 8080)
  -pport int
    	port for pprof server (default 1111)
  -pprof
    	additional server for pprof functionality
  -size string
    	max size for individual files, -1B for infinite size (default "1MB")
```

## Multiple Scribes

After the scribe was successfully produced, I begun wondering what would happen if I had so many log requests that a single scribe wouldn't be able to handle.

It's not possible to start more instances of scribes at different ports, since:

1. the cluster node must know the address and port of the scribe in order to communicate with it, and
1. if two scribes try to write a log line to the same file, it will lock and they will panic.

A **Mediator** was necessary to resolve those two issues. A mediator:

1. keeps track of all instances of scribes,
1. decides which scribe will write to where, and
1. is alone known to the cluster nodes.

So, the first thing that a scribe must do when it starts is to register to the mediator. The mediator keeps track of all scribes registered to him and makes health checks periodically in order to establish which of them are alive and which are not.

This functionality requires the creation of two new gRPC services:
```protobuf
service Register {
  rpc Register (RegisterRequest) returns (RegisterResponse){}
}
service Pinger {
  rpc Ping(PingRequest) returns (PingResponse) {}
}

// PingRequest sends two numbers to mediator
message PingRequest {
  int32 a = 1;
  int32 b = 2;
  string streamerId = 3;
}
// PingResponse returns the mediator's response
message PingResponse {
  int32 res = 2;
}
message RegisterRequest {
  string id = 1;
  string addr = 2;
}
message RegisterResponse {
  string res = 1;
}
```

The Register service is used by the scribes to register themselves to the mediator. They pass in the RegisterRequest their id and their address in the host:port format, and they must get a *Success* response.

The Pinger service is used to make health checks to scribes. It passes two integers and expects their multiplication result.

## Scribe files responsibility algorithm

The mediator keeps track of which files each scribe is responsible for, for any given time. The current algorithm to achieve that is the simplest possible one.

This information is kept on a map that has as a key a character and value a scribe id. The character is compared with the first character of the incoming filename. So, if we have two scribes, the first will be responsible for writing to filenames that begin with *a to r*, including r, and the second scribe from *s to 9*.

After each health check this map is re-generated.

## Log Mediator Usage

Bellow are the flags for the logMediator.

```
Usage of logMediator:
  -ca string
    	certificate authority's certificate
  -crt string
    	host's certificate for secured connections
  -pk string
    	host's private key
  -port int
    	port for mediator server to listen to requests (default 8000)
  -pport int
    	port for pprof server (default 1111)
  -pprof
    	additional server for pprof functionality
```

## TODO

1. add a one-way SSL authentication for the Scribe (or Mediator) only, in addition to the two ways already implemented:

    * an insecure connection (no SSL) and
    * a two-way SSL authentication requiring both the client and the Scribe to have SSL.

1. add more flags for the Scribe and Mediator making them more parameterizable from cl.
1. create a more robust algorithm for load balancing among the Scribes.
1. investigate the use of sync.Map instead of sync.Mutex.
1. keep refactoring.


Get the full code [here](https://github.com/RomanosTrechlis/logScribe)
