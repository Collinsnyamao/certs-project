# Lab 4.19A – Complete and Detailed Guide: Implementing Digital Certificates in OpenSSL

## 🎯 Objective

To understand and demonstrate the process of creating a local Certificate Authority (CA), issuing a digital certificate using OpenSSL, configuring Apache for SSL, and verifying secure HTTPS access using a self-signed certificate.

---

## 🧰 Requirements

- Fedora Linux or similar
- Apache Web Server (`httpd`)
- OpenSSL and `mod_ssl`
- Firefox browser
- Root privileges

---

## 🛠️ Step-by-Step Instructions (with Explanations)

---

### 🔹 Step 1: Install Required Software

```bash
sudo dnf install -y httpd mod_ssl openssl openssl-devel vim firefox
```

Installs required packages for web server (Apache), SSL (mod_ssl), certificate generation (OpenSSL), and browser testing (Firefox).

---

### 🔹 Step 2: Create CA Directory Structure

```bash
sudo mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
sudo touch /etc/pki/CA/index.txt
echo 1000 | sudo tee /etc/pki/CA/serial
```

Sets up the file structure required for a functioning Certificate Authority (CA).

---

### 🔹 Step 3: Configure OpenSSL

Edit OpenSSL config:

```bash
sudo vi /etc/pki/tls/openssl.cnf
```

#### ✅ Ensure/Modify These Sections:

- `[ ca ]`

  ```ini
  default_ca = CA_default
  ```

- `[ CA_default ]`

  ```ini
  dir = /etc/pki/CA
  database = $dir/index.txt
  certificate = $dir/cacert.pem
  private_key = $dir/private/cakey.pem
  serial = $dir/serial
  x509_extensions = usr_cert
  default_md = sha256
  policy = policy_match
  ```

- `[ policy_match ]`

  ```ini
  countryName = match
  stateOrProvinceName = match
  organizationName = match
  organizationalUnitName = optional
  commonName = supplied
  emailAddress = optional
  ```

- `[ req ]`

  ```ini
  default_bits = 2048
  default_md = sha256
  string_mask = utf8only
  distinguished_name = req_distinguished_name
  x509_extensions = v3_ca
  ```

- `[ req_distinguished_name ]` — Add or uncomment:

  ```ini
  countryName_default = US
  stateOrProvinceName_default = Georgia
  localityName_default = Atlanta
  0.organizationName_default = B3
  commonName = localhost
  emailAddress = admin@example.com
  ```

- `[ usr_cert ]` — Ensure this includes:
  ```ini
  basicConstraints = CA:FALSE
  subjectKeyIdentifier = hash
  authorityKeyIdentifier = keyid,issuer
  keyUsage = nonRepudiation, digitalSignature, keyEncipherment
  ```

---

### 🔹 Step 4: Generate CA Certificate

```bash
cd /etc/pki/CA
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -config ../tls/openssl.cnf
```

Creates the self-signed root certificate for your CA.

---

### 🔹 Step 5: Restart Apache

```bash
sudo service httpd restart
```

---

### 🔹 Step 6: Trust the CA in Firefox

1. Open Firefox
2. Go to **Settings → Privacy & Security → View Certificates**
3. Click **Authorities → Import**
4. Choose `/etc/pki/CA/cacert.pem`
5. Check **“Trust this CA to identify websites”**

---

### 🔹 Step 7: Create Server Certificate Request

```bash
cd /etc/pki/CA/private
openssl req -new -keyout newkey.pem -out newreq.pem -days 360 -config ../../tls/openssl.cnf
```

Generates private key and certificate request to be signed.

---

### 🔹 Step 8: Sign the Server Certificate

```bash
sudo bash -c "cat newreq.pem newkey.pem > new.pem"
sudo openssl ca -policy policy_anything -out newcert.pem -config ../../tls/openssl.cnf -infiles new.pem
```

Combines the request and private key, then uses your CA to sign the certificate.

---

### 🔹 Step 9: Configure Apache for SSL

```bash
cd /etc/httpd/conf
sudo mkdir ssl.crt ssl.key
sudo cp /etc/pki/CA/private/newcert.pem ssl.crt/
sudo cp /etc/pki/CA/private/newkey.pem ssl.key/
```

Edit `/etc/httpd/conf.d/ssl.conf`:

```apache
SSLCertificateFile /etc/httpd/conf/ssl.crt/newcert.pem
SSLCertificateKeyFile /etc/httpd/conf/ssl.key/newkey.pem
```

---

### 🔹 Step 10: Restart Apache with SSL

```bash
sudo setenforce 0
sudo service httpd stop
sudo service httpd start
```

---

### 🔹 Step 11: Test in Firefox

Go to:

```
https://localhost
```

- Click **Advanced → Accept the Risk and Continue**
- The secure test page should load

---

### 🔹 Step 12: View Certificate Details

- Click the padlock → More Info → View Certificate
- Check:
  - Certificate Hierarchy (shows your CA and localhost)
  - Signature Algorithm (should be SHA-256)

---

### 🔹 Step 13: Re-enable SELinux

```bash
sudo setenforce 1
getenforce
```

---

## ✅ Result

You have:

- Created a working local CA
- Signed and deployed a server certificate
- Configured Apache for HTTPS
- Verified the connection in Firefox

---

## 🧠 Summary

This lab provides hands-on experience in setting up your own PKI using OpenSSL, managing certs, and enabling SSL with Apache — all foundational security skills.

---
