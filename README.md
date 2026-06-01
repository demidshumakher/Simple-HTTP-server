# Simple-HTTP-server

## What Is This?

Simple-HTTP-server is a small HTTP/1.1 server written in Go for the CodeCrafters HTTP server challenge. It opens a TCP listener on `0.0.0.0:4221`, parses incoming HTTP requests, routes them with a custom regular-expression router, and writes HTTP responses directly to the connection.

The project is intended as a learning implementation of basic HTTP server behavior rather than a production web framework.

## Key Functionality

- Serves `GET /` with a `200 OK` response body.
- Serves `GET /echo/{value}` by returning `{value}` as plain text.
- Serves `GET /user-agent` by returning the request's `User-Agent` header.
- Serves `GET /files/{filename}` by reading a file from the configured directory and returning it as `application/octet-stream`.
- Serves `POST /files/{filename}` by writing the request body to a file in the configured directory and returning `201 Created`.
- Returns `404 Not Found` for unmatched routes.
- Handles accepted TCP connections concurrently with goroutines.

## Architecture & Technology

The server uses a minimal custom architecture built around separate request parsing, routing, response writing, and constants packages. The `app/server.go` entry point wires route handlers into the router and starts the listener.

The project is implemented in Go 1.19 using only the Go standard library. It does not use `net/http` as a server; HTTP requests and responses are parsed and written manually over `net.Conn`.

## Prerequisites

- Go 1.19 or newer
- A shell with access to the repository root

## Installation

Clone the repository and enter the project directory:

```bash
git clone <repository-url>
cd Simple-HTTP-server
```

Download module dependencies:

```bash
go mod download
```

The current module has no third-party dependencies.

## Environment Variables

No environment variables were found in the source code, module files, or repository configuration.

| Name | Required | Description | Example |
|------|----------|-------------|---------|
| None | No | This project does not define environment variables. | N/A |

Runtime configuration is provided through command-line flags:

| Flag | Default | Description | Example |
|------|---------|-------------|---------|
| `-directory` | `/app` | Directory used by the `/files/{filename}` endpoints for file reads and writes. Because filenames are concatenated directly to this value, pass a trailing slash when using a directory path. | `-directory /tmp/http-files/` |

## Running Locally

Start the server:

```bash
go run ./app -directory /tmp/http-files/
```

The server listens on port `4221`:

```text
0.0.0.0:4221
```

Example requests:

```bash
curl -i http://localhost:4221/
curl -i http://localhost:4221/echo/hello
curl -i http://localhost:4221/user-agent
```

File endpoint examples:

```bash
mkdir -p /tmp/http-files
printf 'example content' > /tmp/http-files/example.txt

curl -i http://localhost:4221/files/example.txt
curl -i -X POST http://localhost:4221/files/new.txt --data 'created from curl'
```

## Development

Run the package test command:

```bash
go test ./...
```

Build the server binary:

```bash
go build -o simple-http-server ./app
```

Run the built binary:

```bash
./simple-http-server -directory /tmp/http-files/
```

There are currently no test files in the repository. The `go.mod` file contains CodeCrafters guidance indicating that it should not be edited because their test runner relies on it.

## Build & Deployment

This project builds to a single Go binary:

```bash
go build -o simple-http-server ./app
```

No Dockerfile, deployment manifest, CI configuration, or external runtime service configuration is included in the repository. Deployment consists of running the compiled binary in an environment where TCP port `4221` is available and the configured file directory exists.

## Project Structure

```text
.
├── app/
│   └── server.go              # Main entry point, route registration, server startup
├── pkg/
│   ├── constants/
│   │   └── constants.go       # HTTP method and status message constants
│   ├── request/
│   │   └── request.go         # Manual HTTP request parser
│   ├── response/
│   │   └── response.go        # HTTP response construction and writing
│   └── router/
│       └── router.go          # Regex-based router and TCP listener loop
├── go.mod                     # Go module definition
├── go.sum                     # Module checksum file
└── README.md
```

## Troubleshooting

- `bind: address already in use`: another process is already listening on port `4221`. Stop that process before starting this server.
- `GET /files/{filename}` returns `404 Not Found`: confirm the file exists under the directory passed with `-directory`.
- `POST /files/{filename}` returns `502 Bad Gateway`: confirm the configured directory exists and is writable by the server process.
- File paths do not resolve as expected: include a trailing slash in the `-directory` value, for example `-directory /tmp/http-files/`.

## Additional Notes

- The server listens on a fixed address, `0.0.0.0:4221`.
- The implementation is intentionally minimal and manually handles request parsing and response serialization.
- The repository does not define databases, caches, queues, external APIs, or other external services.
- Responses default to `Content-Type: text/plain`; file downloads use `Content-Type: application/octet-stream`.
