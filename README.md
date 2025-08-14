# self-signed-ssl

```bash
# 認証局発行 (CA)
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes \
  -key ca.key \
  -sha256 \
  -days 3650 \
  -subj "/C=JP/O=Test CA Org/OU=Security Com/CN=testca.com" \
  -addext "subjectAltName = DNS:testca.local, DNS:www.testca.local, IP:127.0.0.1" \
  -out ca.crt

# 証明書発行 (Client)
cat <<EOF | tee client.san
subjectAltName = DNS:testcert.local, DNS:www.testcert.local, IP:127.0.0.1
EOF
openssl genrsa -out client.key 2048
openssl req -new -key client.key \
  -subj "/C=JP/O=Test Org/OU=Security Com/CN=testcert.local/" \
  -out client.csr
openssl x509 -req \
  -extfile client.san \
  -in client.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out client.crt \
  -days 365 -sha256

# がっちゃんこ (P12)
openssl pkcs12 -export \
  -inkey client.key \
  -in client.crt \
  -certfile ca.crt \
  -out client.p12
```