# Self-Signed Certificate Trust Failure

## Error

```
Error: fail to enroll: fail to execute request to fleet-server:
x509: certificate signed by unknown authority
```

## Cause
The self-hosted Fleet Server uses a self-signed TLS certificate (expected for an
internal lab — no public CA was used). Elastic Agent, by default, verifies the server's
cert against a list of trusted CAs, and rejects unknown/self-signed certs.

## Fix used (lab-appropriate)
Added `--insecure` to the enroll/install command, which skips certificate verification
entirely:

```bash
sudo ./elastic-agent install --url=https://<fleet-server-ip>:8220 \
    --enrollment-token=<TOKEN> --insecure
```

## Alternative (not used, more production-correct)
Trust the specific Fleet Server CA cert explicitly instead of disabling verification
globally:

```bash
sudo ./elastic-agent install --url=https://<fleet-server-ip>:8220 \
    --enrollment-token=<TOKEN> \
    --certificate-authorities=/path/to/fleet-server-ca.crt
```

**Note:** `--insecure` is acceptable for a controlled lab environment but should never
be used in production, since it removes protection against a hostile server
impersonating the real Fleet Server.
