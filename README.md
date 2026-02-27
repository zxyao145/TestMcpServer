# TestMcpServer

It exposes a single sample tool over MCP HTTP transport for **generating random numbers**, Primarily used for **testing** the MCP client.

## What it does

- Hosts an MCP server with ASP.NET Core
- Registers one tool: `get_random_number`
- Supports local development over `http://localhost:6025`
- Includes a `Dockerfile` for containerized runs on port `8080`

## Project layout

```text
TestMcpServer.slnx
TestMcpServer/
  Program.cs
  TestMcpServer.csproj
  Tools/
    RandomNumberTools.cs
  Properties/
    launchSettings.json
  TestMcpServer.http
```

## Requirements

- .NET SDK `10.0.100`

The project currently targets `net10.0` and uses `ModelContextProtocol.AspNetCore` version `0.7.0-preview.1`.

## Run locally

From the repository root:

```bash
dotnet run --project TestMcpServer/TestMcpServer.csproj --launch-profile http
```

The default development endpoint is `http://localhost:6025`.

## Run using Docker

### Using Docker hub

```bash
docker run -d -p 8080:8080 --name mcp-test zxyao/test-mcp-server:latest
```

### Local build

Build the image:

```bash
docker build -t test-mcp-server -f TestMcpServer/Dockerfile .
```

Run the container:

```bash
docker run -d -p 8080:8080 --name mcp-test test-mcp-server
```

## Hosting server

```json
{
  "servers": {
    "TestMcpServer": {
      "type": "http",
      "url": "https://mcp.zxyao.net"
    }
  }
}
```

> Rate limiting; return 429 if exceeded

## Available tool

### `get_random_number`

Generates a random integer.

Parameters:

- `min` (`int`, default `0`): inclusive lower bound
- `max` (`int`, default `100`): exclusive upper bound

Implementation: [RandomNumberTools.cs](/mnt/c/Users/zhang/source/repos/TestMcpServer/TestMcpServer/Tools/RandomNumberTools.cs)

## Example MCP client config

```json
{
  "servers": {
    "TestMcpServer": {
      "type": "http",
      "url": "http://localhost:6025"
    }
  }
}
```

## Example request

The repo includes [TestMcpServer.http](/mnt/c/Users/zhang/source/repos/TestMcpServer/TestMcpServer/TestMcpServer.http), which sends a `tools/call` request for `get_random_number`.

Minimal example:

```http
POST / HTTP/1.1
Host: localhost:6025
Accept: application/json, text/event-stream
Content-Type: application/json
MCP-Protocol-Version: 2025-11-25

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_random_number",
    "arguments": {
      "min": 1,
      "max": 10
    }
  }
}
```

## Build and publish

Build:

```bash
dotnet build TestMcpServer.slnx
```

The project is configured to publish as:

- self-contained
- single-file
- native AOT

Configured runtime identifiers:

- `win-x64`
- `win-arm64`
- `osx-arm64`
- `linux-x64`
- `linux-arm64`
- `linux-musl-x64`

## Notes

- Local URLs come from [launchSettings.json](/mnt/c/Users/zhang/source/repos/TestMcpServer/TestMcpServer/Properties/launchSettings.json).
- The container settings come from [Dockerfile](/mnt/c/Users/zhang/source/repos/TestMcpServer/TestMcpServer/Dockerfile).
- An existing template README is present at [TestMcpServer/README.md](/mnt/c/Users/zhang/source/repos/TestMcpServer/TestMcpServer/README.md).
- I was not able to complete `dotnet build TestMcpServer.slnx` in this environment because restore remained at `Determining projects to restore...` for 30 seconds.
