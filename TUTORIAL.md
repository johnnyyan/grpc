# gRPC

gRPC stands for gRPC Remote Procedure Calls. By default, gRPC uses protocol buffers as the Interface Definition Language (IDL) for describing both the service interface and the structure of the payload messages.

## Service Definition Example
```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

## Service Method Types

- Unary RPCs where the client sends a single request to the server and gets a single response back, `rpc SayHello(HelloRequest) returns (HelloResponse);`
- Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back, `rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);`
- Client streaming RPCs where the client writes a sequence of messages and sends them to the server, `rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);`
- Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream, `rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);`

## Notes
### MacOS
- install cmake, `brew install cmake`. then check its vesion `cmake --version`.
- install build tools `brew install autoconf automake libtool`. Skipped `pkg-config` which was already installed. `libtool` probably was already installed too.
- fork grpc repo master only. then clone it to local, `git clone --recurse-submodules --depth 1 --shallow-submodules git@github.com:johnnyyan/grpc.git`.
- set up env, `GRPC_DIR` to `$HOME/.local`. Replace `MY_INSTALL_DIR` with `GRPC_DIR` in the tutorial.

## Linux TODO
- install cmake `$ sudo apt install -y cmake` and `cmake --version`

