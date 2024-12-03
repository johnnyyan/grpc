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

## C++ Notes
### Quick Start
> We __strongly__ encourage you to install gRPC _locally_ — using an appropriately set `CMAKE_INSTALL_PREFIX` — because there is no easy way to uninstall gRPC after you’ve installed it globally.

#### MacOS
- install cmake, `brew install cmake`. then check its vesion `cmake --version`.
- install build tools `brew install autoconf automake libtool shtool`. Skipped `pkg-config` which was already installed. `libtool` probably was already installed too.
- fork grpc repo master only. then clone it to local, `git clone --recurse-submodules --depth 1 --shallow-submodules git@github.com:johnnyyan/grpc.git`. Also ran `git submodule update --init`, no-op. Create `tutorial` branch, `git switch -c tutorial`.
- set up env, `GRPC_DIR` to `$HOME/.local`. Replace `MY_INSTALL_DIR` with `GRPC_DIR` in the tutorial. Add `$GRPC_DIR/bin` to `PATH`.
- build and install gRPC
  - `cd <repo-root>`
  - `mkdir -p cmake/build`
  - `pushd cmake/build`
  - `cmake -DgRPC_INSTALL=ON -DgRPC_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$GRPC_DIR ../..`. Note, originally used flag `-DBUILD_SHARED_LIBS=ON`, but the build failed with `ld: symbol(s) not found for architecture arm64`. It's a known [issue](https://github.com/grpc/grpc/issues/36654).
  - `make -j 4`
  - `make install`
  - `popd`
- build and run [example](https://grpc.io/docs/languages/cpp/quickstart/#build-the-example). Note it is `cmake -DCMAKE_PREFIX_PATH=$GRPC_DIR ../..`, which is intentional.

#### Ubuntu
- install cmake, `sudo apt install -y cmake`, then check its vesion `cmake --version`.
- install build tools, `sudo apt install -y build-essential autoconf libtool pkg-config`.
- clone my grpc fork, `git clone git@github.com:johnnyyan/grpc.git`. `git submodule update --init` to bring in submodules, i.e. third-party dependencies.
- set up env, `GRPC_DIR` to `$HOME/.local`. Replace `MY_INSTALL_DIR` with `GRPC_DIR` in the tutorial. Add `$GRPC_DIR/bin` to `PATH`.
- build and install gRPC
  - `cd <repo-root>`
  - `mkdir -p cmake/build`
  - `pushd cmake/build`
  - `cmake -DgRPC_INSTALL=ON -DgRPC_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$GRPC_DIR -DBUILD_SHARED_LIBS=ON -DABSL_PROPAGATE_CXX_STD=ON ../..`. Note, added `-DABSL_PROPAGATE_CXX_STD=ON` based on warning.
  - `make -j 4`
  - `make install`
  - `popd`
- build and run [example](https://grpc.io/docs/languages/cpp/quickstart/#build-the-example). Note it is `cmake -DCMAKE_PREFIX_PATH=$GRPC_DIR ../..`, see notes below.
  - Have to run `LD_LIBRARY_PATH=~/.local/lib/ make -j 4` cause `grpc_cpp_plugin` missing dependencies, i.e. missing `RUNPATH`.
  - Fixed it by setting `RPATH` for the plugins on Linux, e.g. `set_property(TARGET grpc_cpp_plugin PROPERTY INSTALL_RPATH "$ORIGIN/../${gRPC_INSTALL_LIBDIR}")`.
  - Further fixed all the shared libraries that lack of `RUNPATH` by setting globally `set(CMAKE_INSTALL_RPATH "$ORIGIN;$ORIGIN/../${gRPC_INSTALL_LIBDIR}")`.
  - Using `-DCMAKE_PREFIX_PATH=$GRPC_DIR` can help fix the error during `make -j 4`. But running the applications would still fail for loading shared libraries.\
  `$ ./greeter_server` 
  `./greeter_server: error while loading shared libraries: libupb_wire_lib.so.44: cannot open shared object file: No such file or directory`

### Basics Tutorial
- Fixed a bug to read in the command line arg `--db_path` correctly by using `absl::ParseCommandLine(argc, argv)`.

### Async-API Tutorial
- The tutorial is almost useless! The best way to learn is skim through the tutorial and then read the detailed code directly, `examples/cpp/helloworld/greeter_async_server.cc` then `examples/cpp/helloworld/greeter_async_client.cc`, in which the comments are much more useful!
- Added some debugging info to help understand the process. Note `ServerCompletionQueue->Next(...)` would "Read from the queue, blocking until _an event_ is available or the queue is shutting down." What's the event cycle? What events were emitted?

### Asynchronous Callback API Tutorial

## Q&A

- What is `--grpc_out=.` in `protoc --grpc_out=. ...`?\
  ``$ protoc -I ../../protos --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` ../../protos/route_guide.proto`` will generate grpc code using `protoc-gen-grpc` plugin which is implemented following this [guide](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin/).\
  `$ protoc -I ../../protos --cpp_out=. ../../protos/route_guide.proto` will generate the other protobuf code like `message`, `enum`, etc.

- What is Abseil?\
  [Abseil](https://abseil.io/about/) is an open source collection of C++ libraries drawn from the most fundamental pieces of Google’s internal codebase. For intance, this [flags libray](https://abseil.io/docs/cpp/guides/flags) is extremely useful.
