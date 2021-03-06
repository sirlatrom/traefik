# File Backends

Træfik can be configured with a file.

## Reference

```toml
# Backends
[backends]

  [backends.backend1]

    [backends.backend1.servers]
      [backends.backend1.servers.server0]
        url = "http://10.10.10.1:80"
        weight = 1
      [backends.backend1.servers.server1]
        url = "http://10.10.10.2:80"
        weight = 2
      # ...

    [backends.backend1.circuitBreaker]
      expression = "NetworkErrorRatio() > 0.5"

    [backends.backend1.loadBalancer]
      method = "drr"
      [backends.backend1.loadBalancer.stickiness]
        cookieName = "foobar"

    [backends.backend1.maxConn]
      amount = 10
      extractorfunc = "request.host"

    [backends.backend1.healthCheck]
      path = "/health"
      port = 88
      interval = "30s"

  [backends.backend2]
    # ...

# Frontends
[frontends]

  [frontends.frontend1]
    entryPoints = ["http", "https"]
    backend = "backend1"
    passHostHeader = true
    passTLSCert = true
    priority = 42
    basicAuth = [
      "test:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/",
      "test2:$apr1$d9hr9HBB$4HxwgUir3HP4EsggP/QNo0",
    ]
    whitelistSourceRange = ["10.42.0.0/16", "152.89.1.33/32", "afed:be44::/16"]

    [frontends.frontend1.routes]
      [frontends.frontend1.routes.route0]
        rule = "Host:test.localhost"
      [frontends.frontend1.routes.Route1]
        rule = "Method:GET"
      # ...

    [frontends.frontend1.headers]
      allowedHosts = ["foobar", "foobar"]
      hostsProxyHeaders = ["foobar", "foobar"]
      SSLRedirect = true
      SSLTemporaryRedirect = true
      SSLHost = "foobar"
      STSSeconds = 42
      STSIncludeSubdomains = true
      STSPreload = true
      forceSTSHeader = true
      frameDeny = true
      customFrameOptionsValue = "foobar"
      contentTypeNosniff = true
      browserXSSFilter = true
      contentSecurityPolicy = "foobar"
      publicKey = "foobar"
      referrerPolicy = "foobar"
      isDevelopment = true
      [frontends.frontend1.headers.customRequestHeaders]
        X-Foo-Bar-01 = "foobar"
        X-Foo-Bar-02 = "foobar"
        # ...
      [frontends.frontend1.headers.customResponseHeaders]
        X-Foo-Bar-03 = "foobar"
        X-Foo-Bar-04 = "foobar"
        # ...
      [frontends.frontend1.headers.SSLProxyHeaders]
        X-Foo-Bar-05 = "foobar"
        X-Foo-Bar-06 = "foobar"
        # ...

    [frontends.frontend1.errors]
      [frontends.frontend1.errors.errorPage0]
        status = ["500-599"]
        backend = "error"
        query = "/{status}.html"
      [frontends.frontend1.errors.errorPage1]
        status = ["404", "403"]
        backend = "error"
        query = "/{status}.html"
      # ...

    [frontends.frontend1.ratelimit]
      extractorfunc = "client.ip"
        [frontends.frontend1.ratelimit.rateset.rateset1]
          period = "10s"
          average = 100
          burst = 200
        [frontends.frontend1.ratelimit.rateset.rateset2]
          period = "3s"
          average = 5
          burst = 10
        # ...

    [frontends.frontend1.redirect]
      entryPoint = "https"
      regex = "^http://localhost/(.*)"
      replacement = "http://mydomain/$1"
      permanent = true

  [frontends.frontend2]
    # ...

# HTTPS certificates
[[tls]]
  entryPoints = ["https"]
  [tls.certificate]
    certFile = "path/to/my.cert"
    keyFile = "path/to/my.key"

[[tls]]
  # ...
```

## Configuration mode

You have three choices:

- [Simple](/configuration/backends/file/#simple)
- [Rules in a Separate File](/configuration/backends/file/#rules-in-a-separate-file)
- [Multiple `.toml` Files](/configuration/backends/file/#multiple-toml-files)

To enable the file backend, you must either pass the `--file` option to the Træfik binary or put the `[file]` section (with or without inner settings) in the configuration file.

The configuration file allows managing both backends/frontends and HTTPS certificates (which are not [Let's Encrypt](https://letsencrypt.org) certificates generated through Træfik).

### Simple

Add your configuration at the end of the global configuration file `traefik.toml`:

```toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
    # ...
  [entryPoints.https]
    # ...

[file]

# rules
[backends]
  [backends.backend1]
    # ...
  [backends.backend2]
    # ...

[frontends]
  [frontends.frontend1]
  # ...
  [frontends.frontend2]
  # ...
  [frontends.frontend3]
  # ...

# HTTPS certificate
[[tls]]
  # ...

[[tls]]
  # ...
```

!!! note
    If `tls.entryPoints` is not defined, the certificate is attached to all the `defaultEntryPoints` with a TLS configuration.

!!! note
    Adding certificates directly to the entryPoint is still maintained but certificates declared in this way cannot be managed dynamically.
    It's recommended to use the file provider to declare certificates.

### Rules in a Separate File

Put your rules in a separate file, for example `rules.toml`:

```toml
# traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
    # ...
  [entryPoints.https]
    # ...

[file]
  filename = "rules.toml"
```

```toml
# rules.toml
[backends]
  [backends.backend1]
    # ...
  [backends.backend2]
    # ...

[frontends]
  [frontends.frontend1]
  # ...
  [frontends.frontend2]
  # ...
  [frontends.frontend3]
  # ...

# HTTPS certificate
[[tls]]
  # ...

[[tls]]
  # ...
```

### Multiple `.toml` Files

You could have multiple `.toml` files in a directory (and recursively in its sub-directories):

```toml
[file]
  directory = "/path/to/config/"
```

If you want Træfik to watch file changes automatically, just add:

```toml
[file]
  watch = true
```
