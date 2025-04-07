# Lab 4.19A – Implementing Digital Certificates in OpenSSL

## 📘 Overview

This lab demonstrates how to create a local Certificate Authority (CA), generate and sign a digital certificate using OpenSSL, and configure the Apache HTTP server to use this certificate for SSL/TLS communication. The final result is a secure HTTPS connection to a local Apache server using a self-signed certificate trusted by Firefox.

---

## 🧰 Requirements

- Fedora Linux (or similar RHEL-based distro)
- Root access
- Firefox browser
- Apache Web Server (`httpd`)
- OpenSSL and `mod_ssl` packages

---

## 🛠️ Environment Setup

```bash
sudo dnf install -y httpd mod_ssl openssl openssl-devel vim firefox
sudo systemctl enable httpd
sudo systemctl start httpd
```

---

## 📁 Create CA Directory Structure

```bash
sudo mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
sudo touch /etc/pki/CA/index.txt
echo 1000 | sudo tee /etc/pki/CA/serial
```

---

## 📝 Edit OpenSSL Configuration

Edit `/etc/pki/tls/openssl.cnf` and update these sections:

### [ ca ]

```ini
[ ca ]
default_ca = CA_default
```

### [ CA_default ]

```ini
[ CA_default ]
dir = /etc/pki/CA
certs = $dir/certs
crl_dir = $dir/crl
database = $dir/index.txt
new_certs_dir = $dir/newcerts
certificate = $dir/cacert.pem
serial = $dir/serial
crl = $dir/crl/crl.pem
private_key = $dir/private/cakey.pem
RANDFILE = $dir/private/.rand
x509_extensions = usr_cert
name_opt = ca_default
cert_opt = ca_default
default_days = 365
default_crl_days = 30
default_md = sha256
preserve = no
policy = policy_match
```

### [ policy_match ]

```ini
[ policy_match ]
countryName = match
stateOrProvinceName = match
organizationName = match
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
```

### [ req ]

```ini
[ req ]
default_bits = 1024
default_keyfile = privkey.pem
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca
string_mask = utf8only
```

### [ req_distinguished_name ]

```ini
[ req_distinguished_name ]
countryName_default = US
stateOrProvinceName_default = Georgia
localityName = Atlanta
0.organizationName_default = B3
commonName = Type Yournamehere
emailAddress = your@email.com
```

---

## 🔐 Generate Self-Signed CA Certificate

```bash
cd /etc/pki/CA
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -config ../tls/openssl.cnf
```

- Set a passphrase and confirm certificate details.

---

## 🌐 Restart Apache

```bash
sudo service httpd restart
```

---

## 🦊 Import CA Certificate in Firefox

1. Open Firefox:

   ```bash
   firefox &
   ```

2. Go to **Settings → Privacy & Security → View Certificates**
3. Select the **Authorities** tab and click **Import**
4. Import `/etc/pki/CA/cacert.pem`
5. Check **“Trust this CA to identify websites”**

---

## 🔐 Generate Server Key and Certificate Request

```bash
cd /etc/pki/CA/private
openssl req -new -keyout newkey.pem -out newreq.pem -days 360 -config ../../tls/openssl.cnf
```

> When prompted for **Common Name**, enter `localhost`.

---

## 🧾 Sign the Server Certificate

```bash
sudo bash -c "cat newreq.pem newkey.pem > new.pem"
sudo openssl ca -policy policy_anything -out newcert.pem -config ../../tls/openssl.cnf -infiles new.pem
```

- Enter CA passphrase and confirm signing and committing.

---

## 📂 Move Certificate Files for Apache

```bash
cd /etc/httpd/conf
sudo mkdir ssl.crt ssl.key
sudo cp /etc/pki/CA/private/newcert.pem ssl.crt/
sudo cp /etc/pki/CA/private/newkey.pem ssl.key/
```

---

## ⚙️ Configure Apache for SSL

Edit `/etc/httpd/conf.d/ssl.conf`:

```apache
SSLCertificateFile /etc/httpd/conf/ssl.crt/newcert.pem
SSLCertificateKeyFile /etc/httpd/conf/ssl.key/newkey.pem
```

Save and exit the file.

---

## 🚫 Disable SELinux Temporarily (for lab testing)

```bash
sudo setenforce 0
```

---

## 🔄 Restart Apache

```bash
sudo service httpd stop
sudo service httpd start
```

- Enter passphrase for the key when prompted.

---

## 🔐 Test HTTPS Access

Open Firefox and go to:

```
https://localhost
```

- Click **Advanced → Accept the Risk and Continue** to proceed
- You should see the Apache test page served securely over HTTPS

---

## ✅ Re-enable SELinux

```bash
sudo setenforce 1
getenforce  # Should return: Enforcing
```

---

## 🎉 Result

You have successfully:

- Created a local Certificate Authority
- Signed a certificate for `localhost`
- Configured Apache to use SSL
- Verified HTTPS functionality in Firefox

---

## 📎 Notes

- In real-world environments, use **2048+ bit keys** and `sha256` or higher
- Certificates for public use should always be signed by a **recognized CA**
- Avoid disabling SELinux outside lab or testing scenarios

---
