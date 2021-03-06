---
title: Introduction
---

<p align="center">
    <img src="https://github.com/getsentry/symbolicator/raw/master/artwork/logo.png" width="520" alt="Symbolicator">
    <br />
</p>

Symbolicator is a standalone service that resolves function names, file location
and source context in native stack traces. It can process Minidumps and Apple
Crash Reports. Additionally, Symbolicator can act as a proxy to symbol servers
supporting multiple formats, such as Microsoft's symbol server or Breakpad
symbol repositories.

## Usage

Start the server with:

```shell
$ symbolicator run -c config.yml
```

The configuration file can be omitted. Symbolicator will run with default
settings in this case.

## Configuration

Write this to a file (`config.yml`):

```yaml
cache_dir: "/tmp/symbolicator"
bind: "0.0.0.0:3021"
logging:
  level: "info"
  format: "pretty"
  enable_backtraces: true
metrics:
  statsd: "127.0.0.1:8125"
  prefix: "symbolicator"
```

- `cache_dir`: Path to a directory to cache downloaded files and symbolication
  caches. Defaults to `/data` inside Docker which is already defined as a
  persistent volume, and `null` otherwise, which disables caching. **It is
  strictly recommended to configure caches in production!**
- `bind`: Host and port for HTTP interface.
- `logging`: Command line logging behavior.
    - `level`: Log level, defaults to `info`. Can be one of `off`, `error`,
      `warn`, `info`, `debug`, or `trace`.
    - `format`: The format with which to print logs. Defaults to `auto`. Can be
      one of: `json`, `simplified`, `pretty`, or `auto` (pretty on console,
      simplified on tty).
    - `enable_backtraces`: Whether backtraces for errors should be computed. This
      causes a slight performance hit but improves debuggability. Defaults to
      `true`.
- `metrics`: Configure a statsd server to send metrics to.
    - `statsd`: The host and port to send metrics to. Defaults to `null`, which
      disables metric submission.
    - `prefix`: A prefix for every metric, defaults to `symbolicator`.
- `sentry_dsn`: DSN to a Sentry project for internal error reporting. Defaults
  to `null`, which disables reporting to Sentry.
- `sources`: An optional list of preconfigured sources. If these are configured
  they will be used as default sources for symbolication requests and they will
  be proxied by the symbol proxy if enabled. The format for the sources here
  matches the sources in the HTTP API.
- `symstore_proxy`: Enables or disables the symstore proxy mode. Creates an
  endpoint to download raw symbols from configured sources Symbolicator as if it
  were a `symstore` (Microsoft Symbol Server) compatible server. Defaults to
  `true`.
- `connect_to_reserved_ips`: Allow reserved IP addresses for requests to
  sources. See [Security](#security). Defaults to `false`.

## Security

By default, Symbolicator does not try to download debug files from [reserved IP
ranges](https://en.wikipedia.org/wiki/Reserved_IP_addresses). This ensures that
no unintended connections are made to internal systems when source configuration
is passed in from an untrusted source.

To allow internal connections, set `connect_to_reserved_ips` to `true`.

An exception from this rule is the `"sentry"` source type. Sentry is expected to
run within the same network as Symbolicator, which is why it is exempt by
default.
