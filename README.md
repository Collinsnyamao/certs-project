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

**Explanation:** Installs the Apache server, SSL support, OpenSSL for crypto operations, and Firefox for browser testing.

---

### 🔹 Step 2: Create CA Directory Structure

```bash
sudo mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
sudo touch /etc/pki/CA/index.txt
echo 1000 | sudo tee /etc/pki/CA/serial
```

**Explanation:** Sets up the file structure required for a functioning local Certificate Authority.

---

### 🔹 Step 3: Configure OpenSSL

Edit OpenSSL config:

```bash
sudo vi /etc/pki/tls/openssl.cnf
```

Update or ensure the following sections match:

- `[ ca ]` and `[ CA_default ]` for directory paths and key/cert files
- `[ policy_match ]` for certificate field requirements
- `[ req ]` to define the request format
- `[ req_distinguished_name ]` for default identity values
- `default_md = sha256`
- `string_mask = utf8only`

---

### 🔹 Step 4: Generate CA Certificate

```bash
cd /etc/pki/CA
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -config ../tls/openssl.cnf
```

**Explanation:** This is your root CA certificate used to sign other certificates.

---

### 🔹 Step 5: Restart Apache

```bash
sudo service httpd restart
```

---

### 🔹 Step 6: Trust the CA in Firefox

1. Launch Firefox:
   ```bash
   firefox &
   ```
2. Go to **Settings → Privacy & Security → View Certificates**
3. Click **Authorities → Import**
4. Choose: `/etc/pki/CA/cacert.pem`
5. Check **“Trust this CA to identify websites”**

---

### 🔹 Step 7: Create Server Certificate Request

```bash
cd /etc/pki/CA/private
openssl req -new -keyout newkey.pem -out newreq.pem -days 360 -config ../../tls/openssl.cnf
```

**Explanation:** Generates a private key and certificate signing request (CSR) for the server.

---

### 🔹 Step 8: Sign the Server Certificate

```bash
sudo bash -c "cat newreq.pem newkey.pem > new.pem"
sudo openssl ca -policy policy_anything -out newcert.pem -config ../../tls/openssl.cnf -infiles new.pem
```

**Explanation:** The CA signs the request to issue a valid certificate (`newcert.pem`).

---

### 🔹 Step 9: Configure Apache to Use SSL

```bash
cd /etc/httpd/conf
sudo mkdir ssl.crt ssl.key
sudo cp /etc/pki/CA/private/newcert.pem ssl.crt/
sudo cp /etc/pki/CA/private/newkey.pem ssl.key/
```

Update `/etc/httpd/conf.d/ssl.conf`:

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

### 🔹 Step 11: Test HTTPS in Browser

1. Open Firefox
2. Visit: `https://localhost`
3. Click **Advanced → Accept the Risk and Continue**
4. Page should load securely

---

### 🔹 Step 12: View Certificate Details

1. Click the padlock icon in Firefox
2. Click **“Connection Secure” → “More Information” → “View Certificate”**
3. Note:
   - **Certificate Hierarchy**: should show your CA and `localhost`
   - **Signature Algorithm**: typically `sha256WithRSAEncryption`

---

### 🔹 Step 13: Re-enable SELinux

```bash
sudo setenforce 1
getenforce  # Should return: Enforcing
```

---

## ✅ Result

You have:

- Created a local CA
- Signed a server certificate
- Configured Apache for SSL
- Verified HTTPS with trusted CA in Firefox

---

## 🧠 Summary

This lab provides practical experience with:

- Public Key Infrastructure (PKI)
- OpenSSL certificate generation and signing
- Configuring HTTPS on Apache
- Certificate trust management in browsers

---
