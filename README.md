# directory-mcp-server

A standalone C++20 Model Context Protocol (MCP) server for filesystem directory operations.

The server exposes five tools over MCP 2026-07-28:

| Tool | Description |
|---|---|
| `create_directory` | Creates a directory, including missing parent directories. |
| `list_directory` | Lists the immediate entries of a directory. |
| `iterate_directories_recursively` | Recursively enumerates directory entries with bounded traversal. |
| `delete_directory` | Recursively deletes a directory and its contents. |
| `get_last_modified_file` | Finds the most recently modified regular file beneath a directory. |

**This server accesses the host filesystem with the permissions of the process running it.** In particular, `delete_directory` is destructive. Run it only with a client and filesystem scope you trust.

## Dependencies

The MCP protocol, server, transport, process, JSON, and HTTP implementations are provided by [`pooriayousefi/mcpxx`](https://github.com/pooriayousefi/mcpxx), included as a pinned Git submodule at `external/mcpxx`.

The submodule is pinned to commit `8d3e971072c263dd5132c5b48e44c15926d21190`. This keeps builds reproducible and avoids copying shared MCP implementation files into each server repository.

## Requirements

- C++20 compiler (Clang or GCC)
- POSIX threads
- Git, including submodule support

## Clone

Clone this repository with its dependency:

```sh
git clone --recurse-submodules https://github.com/pooriayousefi/directory-mcp-server.git
cd directory-mcp-server
```

If you cloned without submodules:

```sh
git submodule update --init --recursive
```

## Build

Using the included Makefile:

```sh
make
```

Or compile directly:

```sh
mkdir -p bin
clang++ -std=c++20 -O2 -pthread \
  -I external/mcpxx/include \
  src/directory_mcp_tool_server.cpp \
  -o bin/directory_mcp_tool_server
```

Use `g++` instead of `clang++` if preferred. On Windows with MinGW, add `-lws2_32`.

## Run

The default transport is stdio, intended for MCP clients that launch the server as a child process:

```sh
./bin/directory_mcp_tool_server
```

To run the Streamable HTTP transport:

```sh
./bin/directory_mcp_tool_server http 127.0.0.1 8094
```

The server exposes the MCP endpoint at `/mcp`. The default HTTP port is `8094`.

Keep HTTP access bound to loopback unless you have independently configured appropriate network controls. The HTTP transport is not an authentication boundary.

## Integration

Configure an MCP client to launch `bin/directory_mcp_tool_server` using stdio. The server advertises its tool names, descriptions, and JSON input schemas through standard MCP discovery.

## License

Apache-2.0. See [LICENSE](LICENSE).
