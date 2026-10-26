# hdxml

Read-only SystemVerilog RTL analyzer for [autowire](https://github.com/rede97/autowire).
It scans `.v` / `.sv` sources and writes an RtlIndex XML directory (one XML per
source file plus `index.xml`) with module hierarchy, parameters, ports, and
instances. No subcommands.

## Download

Prebuilt binaries are on the [releases page](https://github.com/rede97/hdxml/releases):

| Asset | Platform |
|---|---|
| `hdxml-linux-x64.tar.xz` | Linux x86_64, glibc 2.17 (CentOS 7+) |
| `hdxml-macos-arm64.tar.xz` | macOS Apple silicon |
| `hdxml-macos-x64.tar.xz` | macOS Intel |
| `hdxml-windows-x64.zip` | Windows x64 |

autowire pins one hdxml version and downloads it from here; put `hdxml` on
`PATH` or point autowire at it with `--hdxml`, `[analysis] hdxml_bin`, or
`$HDXML_BIN`.

## Build

```bash
cargo build --release
cargo test
```

CLI and RtlIndex format: [`docs/hdxml/`](https://github.com/rede97/autowire/tree/main/docs/hdxml)
in the autowire repository. Developer notes: [`AGENTS.md`](AGENTS.md).
