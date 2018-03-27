# Certify
[![pipeline status](https://gitlab.com/jbrandhorst/certify/badges/master/pipeline.svg)](https://gitlab.com/jbrandhorst/certify/commits/master)
[![GoDoc](https://godoc.org/github.com/johanbrandhorst/certify?status.svg)](https://godoc.org/github.com/johanbrandhorst/certify)
[![Go Report Card](https://goreportcard.com/badge/github.com/johanbrandhorst/certify)](https://goreportcard.com/report/github.com/johanbrandhorst/certify)

Certify allows easy automatic certificate distribution and maintenance.
Certificates are requested as TLS connections
are made, courtesy of the `GetCertificate` and `GetClientCertificate`
`tls.Config` hooks. Certificates are optionally cached.

## Issuers

Certify exposes an `Issuer` interface which is used to allow switching
between issuer backends.

Currently implemented issuers:

- Vault PKI Secrets Engine

## Usage

Create an issuer:
```go
issuer := &certify.VaultIssuer{
    VaultURL: &url.URL{
        Scheme: "https",
        Host: "my-local-vault-instance.com"
    },
    Token:     "myVaultToken",
    Role:      "myVaultRole",
}
```

Create a Certify:
```go
c := &certify.Certify{
    CommonName: "MyServer.com",
    Issuer: issuer,
    // It is recommended to use a cache.
    Cache: certify.NewMemCache(),
    // It is recommended to set a RenewThreshold.
    // Refresh cached certificates when < 24H left before expiry.
    RenewThreshold: 24*time.Hour,
}
```

Use in your TLS Config:
```go
tlsConfig := &tls.Config{
    GetCertificate: c.GetCertificate,
}
```

That's it! Both server-side and client-side certificates
can be generated:

```go
tlsConfig := &tls.Config{
    GetClientCertificate: c.GetClientCertificate,
}
```

For an end-to-end example using gRPC with mutual TLS authentication,
see the tests.
