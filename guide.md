# 🎓 Implementing Digital Certificates with OpenSSL & Apache

---

## Slide 1: Introduction

**Objective:**
Understand and demonstrate how digital certificates and Certificate Authorities work using OpenSSL and Apache.

---

## Slide 2: Fedora Apache Test Page

![Fedora Test Page](./Screenshot From 2025-04-07 13-56-26.png)

This shows Apache is installed and running properly on Fedora.

---

## Slide 3: Installing OpenSSL

![Installing OpenSSL](./Screenshot From 2025-04-07 13-57-36.png)

Install OpenSSL to enable certificate generation.

---

## Slide 4: Verifying Web Server is Running

![Apache Running](./Screenshot From 2025-04-07 13-58-22.png)

Use `systemctl` to enable and start the Apache HTTP server.

---

## Slide 5: CA Directory Structure

![Creating CA Directory](./Screenshot From 2025-04-07 14-01-00.png)

Create the folder structure used by the Certificate Authority (CA) and initialize it.

---

## Slide 6: Editing openssl.cnf

![Editing openssl.cnf - Part 1](./Screenshot From 2025-04-07 14-38-37.png)
![Editing openssl.cnf - Part 2](./Screenshot From 2025-04-07 14-39-05.png)

Customize OpenSSL configuration to reflect CA and request policies.

---

## Slide 7: Creating the CA Certificate

![Creating CA Cert](./Screenshot From 2025-04-07 14-41-41.png)

This generates a self-signed certificate for the CA.

---

## Slide 8: Providing Certificate Information

![Inputting DN Info](./Screenshot From 2025-04-07 14-42-24.png)

Details like organization, common name, and email are entered here.

---

## Slide 9: Verifying Apache Status

![Apache Status](./Screenshot From 2025-04-07 14-43-28.png)

Confirm that Apache is running successfully.

---

## Slide 10: Importing Certificate in Firefox

![Firefox Certificate Import](./Screenshot From 2025-04-07 14-45-11.png)

Import your CA certificate into Firefox to trust your local server.

---

## Slide 11: Conclusion

- Set up a CA and signed certificates
- Configured Apache to use them
- Verified secure connection with Firefox

This forms the foundation of web-based trust and encryption!
