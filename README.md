# OpenSSL PKI Lab: Create a Root CA and Issue a Server Certificate

This lab demonstrates how to:

1. Create a Root Certificate Authority (CA)
2. Generate a server private key
3. Create a Certificate Signing Request (CSR)
4. Define certificate extensions
5. Sign the CSR with the CA
6. Generate a trusted server certificate

---

# PKI Workflow

```text
CA Private Key
      ↓
CA Certificate
      ↓
Server Private Key
      ↓
Server CSR
      ↓
Certificate Extensions
      ↓
Signed Server Certificate
```

---

# Step 1 - Generate the CA Private Key

Generate a 2048-bit RSA private key protected with a passphrase.

```bash
openssl genrsa -des3 -out ca1.key 2048
```

Example output:

```text
Generating RSA private key, 2048 bit long modulus
Enter pass phrase for ca1.key:
```

Result:

```text
ca1.key
```

Purpose:

- Root CA private key
- Used to sign certificates
- Must be kept secure

---

# Step 2 - Create the Root CA Certificate

Create a self-signed X.509 certificate.

```bash
openssl req -x509 -new -nodes \
-key ca1.key \
-sha256 \
-days 900 \
-out ca1.pem
```

Enter required certificate information:

```text
Country Name
State Name
Organization Name
Common Name
```

Result:

```text
ca1.pem
```

Purpose:

- Root CA certificate
- Distributed to clients that need to trust certificates issued by this CA

---

# Step 3 - Generate the Server Private Key

Generate the private key for the web server.

```bash
openssl genrsa -out www.mylocalsite.local 2048
```

Result:

```text
www.mylocalsite.local
```

Purpose:

- Server's private key
- Used during TLS/HTTPS handshakes
- Must remain secret

---

# Step 4 - Create a Certificate Signing Request (CSR)

Generate a CSR using the server private key.

```bash
openssl req -new \
-key www.mylocalsite.local \
-out www.mylocalsite.csr
```

Provide certificate details when prompted.

Example:

```text
Common Name (CN):
www.mylocalsite.local
```

Result:

```text
www.mylocalsite.csr
```

Purpose:

- Requests a certificate from the CA
- Contains the server's public key
- Does NOT contain the private key

---

# Step 5 - Create the Extension File

Create a file named:

```bash
nano www.mylocalsite.ext
```

Add the following content:

```ini
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage=digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
subjectAltName=@alt_names

[alt_names]
DNS.1=www.mylocalsite.local
```

Purpose:

- Defines certificate usage
- Specifies Subject Alternative Names (SAN)
- Required for modern browser compatibility

---

# Step 6 - Sign the CSR Using the CA

Create the final server certificate.

```bash
openssl x509 -req \
-in www.mylocalsite.csr \
-CA ca1.pem \
-CAkey ca1.key \
-CAcreateserial \
-out www.mylocalsite.crt \
-days 365 \
-sha256 \
-extfile www.mylocalsite.ext
```

Enter the CA private key passphrase when prompted:

```text
Enter pass phrase for ca1.key:
```

Result:

```text
www.mylocalsite.crt
```

Purpose:

- Final signed server certificate
- Can be installed on a web server

---

# Final Files

```text
ca1.key
```

- Root CA Private Key

```text
ca1.pem
```

- Root CA Certificate

```text
www.mylocalsite.local
```

- Server Private Key

```text
www.mylocalsite.csr
```

- Certificate Signing Request

```text
www.mylocalsite.ext
```

- Certificate Extensions

```text
www.mylocalsite.crt
```

- Server Certificate

---

# Verify the Certificate

Display certificate details:

```bash
openssl x509 -in www.mylocalsite.crt -text -noout
```

Verify certificate against the CA:

```bash
openssl verify -CAfile ca1.pem www.mylocalsite.crt
```

Expected output:

```text
www.mylocalsite.crt: OK
```

---

# Install on a Web Server

Typical HTTPS deployment:

```text
Server Certificate:
www.mylocalsite.crt

Server Private Key:
www.mylocalsite.local

CA Certificate:
ca1.pem
```

TLS clients will:

1. Receive the server certificate
2. Validate the CA signature
3. Verify the hostname
4. Confirm the server owns the private key
5. Establish an encrypted HTTPS connection

---

# Key Concepts

```text
Private Key → CSR → Certificate
```

```text
genrsa      → Generate Private Key
req -new    → Generate CSR
req -x509   → Create Self-Signed CA Certificate
x509 -req   → Sign CSR and Generate Certificate
```

<img width="800" height="347" alt="image" src="https://github.com/user-attachments/assets/26deb634-4b6a-416a-a640-7388136e5422" />
