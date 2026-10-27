# OpenSSL

## Download a certificate from an address

```shell
openssl s_client -showcerts -connect minio.local.com:443 -servername minio.local.com < /dev/null 2>/dev/null | openssl x509 -outform PEM > minio.local.com.crt
```