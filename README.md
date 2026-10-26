# onesimus-proto

Protocol Buffer definitions for the Onesimus Director gRPC interface: the
contract between the Onesimus Director (`odir`), the Onesimus
console (`oconsole`, including its MCP server) and the Onesimus GUI.

The gRPC interface complements the classic console protocol: the Onesimus
Director stays wire-compatible with the Bareos console protocol (bconsole,
Bareos WebUI), while this repository defines the Onesimus gRPC API.

Protobuf packages: `onesimus` (custom options), `onesimus.director`,
`onesimus.director.commands`, `onesimus.director.api`, `onesimus.ccc`.

## Files

| File | Description |
|------|-------------|
| `director_service.proto` | Director gRPC service `onesimus.director.Director` (GetServerInfo, Login, ExecuteCommand, ListCommands, InteractiveSession) |
| `director_api.proto` | Typed Director API (`onesimus.director.api.DirectorApi`): one RPC per command/subcommand with structured results |
| `director_commands.proto` | 88 command definitions with parameter schemas (`onesimus.director.commands`) |
| `onesimus_options.proto` | Custom protobuf options for command metadata (`(onesimus.command_name)`, `(onesimus.help_text)`, `(onesimus.usage)`, `(onesimus.bare_flag)`, …) |
| `plugin_ccc.proto` | Cross-cutting plugin interfaces shared by FD/SD/DIR (`onesimus.ccc`) |

## Typed Director API (`director_api.proto`)

`DirectorApi` runs on the same gRPC port as `Director` and uses the same
login (session token in the metadata header `x-onesimus-session`) and the
same console ACLs. Requests are the `*Command` messages of
`director_commands.proto` where they carry all parameters of a command form,
otherwise new `*Command` messages in `director_api.proto` (with the
`onesimus_options.proto` options). Responses are `*Response` messages built
from `*Info` records whose field names are the keys of the Director's
API 2 JSON output (`.api 2`); values without a typed field are kept in
`extra_fields`.

- Read RPCs (list/llist, status, show, dot listings, BVFS/OVFS queries) are
  implemented by odir in every build.
- Write, session and restore-tree RPCs are implemented only in the
  odir development build (cargo feature `grpc-development`); the
  production build answers `UNIMPLEMENTED`. Each runs one non-interactive
  console command and returns `ActionResponse` (text, errors, job ids and
  the typed job/volume/pool records after the change). Destructive RPCs
  (Relabel, Prune*, Purge*, Delete*, Truncate) require `confirm = true`.
- Interactive restore: `StartRestoreSession` builds the file tree and
  returns a `session_id` (bound to the login); the `RestoreTree*` RPCs are
  the file selection commands (cd, ls, mark, …), `RestoreTreeDone` starts
  the restore job.
- `JobStatus`, `JobType` and `JobLevel` enum numbers are the ASCII codes of
  the catalog characters (`JOB_STATUS_TERMINATED = 84 = 'T'`).
- Imports `google/protobuf/struct.proto` (well-known types, e.g. Debian
  package `libprotobuf-dev`).

Example (grpcurl, after `Login`):

```bash
grpcurl -insecure -import-path proto -proto director_api.proto \
  -H "x-onesimus-session: $TOKEN" localhost:10101 \
  onesimus.director.api.DirectorApi/GetVersion
```

Documentation (German): `docs/wiki/grpc-api.md` in the Onesimus Director
repository.

## Usage

### As Git submodule

```bash
git submodule add git@github.com:Beerlesklopfer/onesimus-proto.git proto
```

### In Rust (tonic-build)

```rust
// build.rs
tonic_build::configure()
    .build_server(true)
    .build_client(true)
    .compile_protos(
        &["proto/director_service.proto"],
        &["proto/"],
    )
    .unwrap();

// typed API: servers that implement only part of it can let the generated
// default methods answer UNIMPLEMENTED
tonic_build::configure()
    .build_server(true)
    .build_client(true)
    .generate_default_stubs(true)
    .compile_protos(&["proto/director_api.proto"], &["proto/"])
    .unwrap();

// generated modules: onesimus::director, onesimus::director::api, ...
```

### In C++ (CMake)

```cmake
find_package(Protobuf REQUIRED)
find_package(gRPC CONFIG REQUIRED)
add_library(onesimus_proto
  director_service.proto director_commands.proto
  director_api.proto onesimus_options.proto)
protobuf_generate(TARGET onesimus_proto LANGUAGE cpp)
protobuf_generate(TARGET onesimus_proto LANGUAGE grpc
  GENERATE_EXTENSIONS .grpc.pb.h .grpc.pb.cc
  PLUGIN "protoc-gen-grpc=$<TARGET_FILE:gRPC::grpc_cpp_plugin>")
```

Generated C++ namespaces follow the packages (`onesimus::director`,
`onesimus::director::api`, …); the options file generates
`onesimus_options.pb.h`.

## License

GNU Affero General Public License v3.0 only (`AGPL-3.0-only`); see [LICENSE](LICENSE).
Parts derived from Bareos keep their original copyright notices.

Maintainer: Jörg Bernau <Joerg.Bernau@onesimus.io>.
