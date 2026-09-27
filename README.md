# TLS Checker

TLS Checker is a Go command-line tool that checks hosts for DNS resolution, TLS connectivity, certificate details, ALPN negotiation, HTTP/2 readiness, and (optionally) ASN information. It reads targets from a text file and checks them concurrently.

## Requirements

- Go 1.26.1 or newer
- Network access to the targets being checked

## Build and run

Run the program directly from the repository. This exercises the CLI's `main` entry point:

```sh
go run . -i example_urls.txt
```

Or build a local executable:

```sh
go build -o tls_checker .
./tls_checker -i example_urls.txt
```

On Windows, run the generated `tls_checker.exe` instead. To build the release binaries for all supported platforms, use `make build` (requires Make and a shell with the utilities used by the Makefile).

## Input file

By default, the program reads `example_urls.txt` from the current directory. Put one host, IP address, or URL on each line. Blank lines and lines beginning with `#`, `//`, `;`, or `--` are ignored. Duplicate host and port pairs are checked once. URLs without an explicit port use the default port (443).

```text
# Production endpoints
example.com
https://www.google.com
https://example.org:8443/health
192.0.2.10
```

## CLI options

| Option | Default | Description |
| --- | --- | --- |
| `-i <file>` | `example_urls.txt` | Input file containing hosts, IPs, or URLs |
| `-t <count>` | `12` | Number of concurrent workers (values below 1 are treated as 1) |
| `--timeout <duration>` | `5s` | Per-connection timeout, such as `3s` or `1m` |
| `--retries <count>` | `3` | Retries for retryable failures |
| `-o <file>` | — | Write detailed results to this file |
| `-v` | off | Enable debug output |
| `--no-asn` | off | Disable Team Cymru ASN lookups |
| `--port <port>` | `443` | Default port for targets without an explicit port |
| `-version` | — | Print the program version and exit |
| `-update` | — | Check for and install the latest release |

## Examples

Check the sample list with 16 workers, a five-second timeout, and two retries:

```sh
go run . -i example_urls.txt -t 16 --timeout 5s --retries 2
```

Check a custom list without ASN lookups and save output:

```sh
go run . -i hosts.txt --no-asn -o results.txt
```

Show the version or request an update from an installed binary:

```sh
./tls_checker -version
./tls_checker -update
```

## Results and exit status

Each result reports the resolved IP, connection round-trip time, certificate common name and SANs, ASN (unless disabled), TLS version, ALPN, HTTP/2 probe status, and certificate validity. Outcomes are grouped as:

- **Full:** TLS 1.3, ALPN `h2`, and a successful HTTP/2 probe.
- **Success:** TLS 1.3, even if HTTP/2 is not available.
- **Partial:** TLS connected using a version older than TLS 1.3.
- **Failure:** A DNS, timeout, TLS, or other check error.

The process exits with status `0` when every target succeeds, `2` when at least one target check fails, and `1` for startup errors such as an unreadable input file or no usable targets.

## Development checks

Run the Go test suite and static analysis from the repository root:

```sh
go test ./...
go vet ./...
```

## Installation (macOS/Linux)

The install script downloads the latest release and installs it:

```sh
curl -fsSL https://raw.githubusercontent.com/sinnet3000/tls_checker/main/scripts/install.sh | bash
```

To build and install from source into `~/.local/bin`:

```sh
make install
```

## License

This project is licensed under the AGPL-3.0 License. See [LICENSE](LICENSE) for details.
