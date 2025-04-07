# Lab 4.19A – Detailed Guide: Implementing Digital Certificates in OpenSSL

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

**What it does:**

- Installs the Apache web server
- Installs SSL module for Apache
- Installs OpenSSL for certificate operations
- Installs Firefox for testing HTTPS

---

### 🔹 Step 2: Create Certificate Authority (CA) Directory Structure

```bash
sudo mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
sudo touch /etc/pki/CA/index.txt
echo 1000 | sudo tee /etc/pki/CA/serial
```

**Explanation:**

- Sets up directories and files used by OpenSSL to manage CA operations, including issued certs, revocation lists, and serial tracking.

---

### 🔹 Step 3: Configure OpenSSL

Edit the file:

```bash
sudo vi /etc/pki/tls/openssl.cnf
```

**What to change and why:**

- `default_md = sha256` — uses secure hashing
- `dir = /etc/pki/CA` — defines CA base directory
- `certificate` and `private_key` — paths to CA cert and key
- `policy_match` — sets rules for certificate info validation

This tells OpenSSL where to read/write CA data and what rules to enforce.

---

### 🔹 Step 4: Generate Self-Signed CA Certificate

```bash
cd /etc/pki/CA
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -config ../tls/openssl.cnf
```

**Explanation:**

- Creates the CA’s own certificate (`cacert.pem`) and private key (`cakey.pem`)
- Required for signing other certificates

---

### 🔹 Step 5: Restart Apache

```bash
sudo service httpd restart
```

**Why:**

- Ensures Apache is running and ready to serve content over HTTP/HTTPS

---

### 🔹 Step 6: Trust the CA in Firefox

1. Open Firefox
2. Go to **Settings → Privacy & Security → View Certificates**
3. Under **Authorities**, click **Import**
4. Select `/etc/pki/CA/cacert.pem`
5. Check **"Trust this CA to identify websites"**

**Why:**

- Without trusting this CA, Firefox will treat all certs signed by it as untrusted

---

### 🔹 Step 7: Create Server Certificate Request

```bash
cd /etc/pki/CA/private
openssl req -new -keyout newkey.pem -out newreq.pem -days 360 -config ../../tls/openssl.cnf
```

**What it does:**

- Generates a private key and CSR (certificate signing request)
- This request is what the CA will sign

---

### 🔹 Step 8: Sign the Server Certificate

```bash
sudo bash -c "cat newreq.pem newkey.pem > new.pem"
sudo openssl ca -policy policy_anything -out newcert.pem -config ../../tls/openssl.cnf -infiles new.pem
```

**Explanation:**

- Combines CSR and key
- Signs the certificate using the CA’s private key
- `newcert.pem` is now a valid cert trusted by your local CA

---

### 🔹 Step 9: Configure Apache to Use SSL

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

**Explanation:**

- Apache needs to know where your cert and key are to enable SSL

---

### 🔹 Step 10: Restart Apache with SSL

```bash
sudo setenforce 0
sudo service httpd stop
sudo service httpd start
```

**Explanation:**

- Temporarily disables SELinux to avoid access errors
- Restarts Apache so it picks up the new SSL config

---

### 🔹 Step 11: Test the HTTPS Connection

Open Firefox and navigate to:

```
https://localhost
```

- You may get a security warning (expected)
- Click “Advanced → Accept the Risk and Continue”
- The Apache welcome page should load over HTTPS

---

### 🔹 Step 12: Re-enable SELinux

```bash
sudo setenforce 1
getenforce
```

**Why:**

- Best practice is to keep SELinux enforcing unless troubleshooting

---

## 🧾 Result

You have:

- Created a working local Certificate Authority
- Generated and signed a server certificate
- Configured Apache to use SSL with that certificate
- Verified secure HTTPS access via browser

---

## 🧠 Summary

- Digital certificates provide secure communication and identity verification
- OpenSSL enables you to build a PKI environment from scratch
- Apache supports SSL easily when provided with valid cert and key files

---
