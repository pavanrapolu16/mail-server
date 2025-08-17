# Personal Mail Server Setup (Postfix + Dovecot) on Cloud

A project documenting the process of setting up a personal mail server using **Postfix** and **Dovecot** on cloud servers (AWS and Oracle). This setup allows sending and receiving emails securely with TLS/SSL.

---

## Table of Contents

* [Overview](#overview)
* [Protocols Used](#protocols-used)
* [Cloud Setup](#cloud-setup)
* [Installation & Configuration](#installation--configuration)
* [DNS Records](#dns-records)
* [Difficulties Encountered](#difficulties-encountered)
* [Achievements](#achievements)
* [Email Flow](#email-flow)
* [License](#license)

---

## Overview

This project demonstrates building a personal mail server from scratch, including:

* Sending/receiving emails using SMTP.
* Securing communication with TLS/SSL.
* Managing email delivery in Linux mailbox (`/var/mail`).
* Configuring Postfix and Dovecot for cloud deployment.

The project also includes lessons learned from **AWS port restrictions** and **Oracle Cloud setup**.

---

## Protocols Used

* **SMTP (Simple Mail Transfer Protocol):** Sending emails between client and server.
* **IMAP / POP3:** Accessing emails stored on the server (not fully configured yet).
* **TLS/SSL:** Securing SMTP communication (ports 465/587).
* **HTTP/HTTPS:** Web access through Nginx (ports 80/443).

---

## Cloud Setup

1. **AWS EC2**

   * Initial attempt blocked due to **port 25 restriction**.
   * Learned AWS security groups and inbound/outbound rules.

2. **Oracle Cloud**

   * Port 25 was open → allowed SMTP testing.
   * Gained experience with VCN, subnets, and hostnames.
   * Realized Oracle has some restrictions similar to AWS.

3. **Final Setup**

   * Returned to AWS with proper configuration.
   * Used Postfix + Dovecot + TLS.
   * Emails successfully delivered to `/var/mail/adminuser`.

---

## Installation & Configuration

1. **Install Postfix and Dovecot**

   ```bash
   sudo apt update
   sudo apt install postfix dovecot-core dovecot-imapd
   ```

2. **Generate TLS Certificates**

   ```bash
   sudo mkdir -p /etc/ssl/sthara
   sudo openssl req -x509 -nodes -days 365 \
     -newkey rsa:2048 \
     -keyout /etc/ssl/sthara/sthara.key \
     -out /etc/ssl/sthara/sthara.crt
   ```

3. **Configure Postfix (`/etc/postfix/main.cf`)**

   * Enable submission ports 587/465.
   * Set TLS parameters:

     ```conf
     smtpd_tls_cert_file=/etc/ssl/sthara/sthara.crt
     smtpd_tls_key_file=/etc/ssl/sthara/sthara.key
     smtpd_tls_security_level=may
     smtpd_sasl_auth_enable=yes
     ```

4. **Reload Postfix**

   ```bash
   sudo postfix reload
   ```

5. **Verify listening ports**

   ```bash
   sudo netstat -tulnp | grep master
   ```

   * Expected: `25, 465, 587` for SMTP, `80, 443` for Nginx.

6. **Check emails**

   ```bash
   sudo cat /var/mail/adminuser
   ```

---

## DNS Records

To properly send and receive emails, configure the following DNS records for your domain `sthara.fun`:

* **MX Record:**

  * `@` → `mail.sthara.fun` (priority 10)
  * Directs incoming mail to your server.

* **A Record:**

  * `mail.sthara.fun` → `<server IP>`
  * Maps domain to your server’s public IP.

* **SPF Record (TXT):**

  * `v=spf1 mx ~all`
  * Authorizes your server to send emails on behalf of the domain.

* **DKIM Record (TXT):**

  * Generated from Postfix/Dovecot setup.
  * Ensures emails are not tampered with.

* **DMARC Record (TXT):**

  * `v=DMARC1; p=none; rua=mailto:admin@sthara.fun`
  * Provides reporting and prevents spoofing.

These records help email providers like Gmail recognize your server as legitimate and prevent messages from going to spam.

---

## Difficulties Encountered

* **Port Restrictions**

  * AWS blocked port 25 initially.
* **Library Deprecation**

  * Node.js libraries for mail servers became outdated → switched to Postfix + Dovecot.
* **Postfix Syntax Errors**

  * Incorrect `main.cf` entries caused `missing '=' after attribute` errors.
* **Permissions**

  * Accessing `/home/adminuser/` required sudo and proper understanding of Linux permissions.
* **Cloud Learning Curve**

  * Understanding VCN, firewall rules, and FQDN in Oracle Cloud.

---

## Achievements

* Successfully received emails from Gmail to `/var/mail/adminuser`.
* Configured TLS encryption for secure SMTP communication.
* Learned cloud server setup, firewall rules, and hostname configuration.
* Built understanding of email protocols and Postfix/Dovecot configuration.
* Gained practical troubleshooting experience for real-world mail servers.

---

## Email Flow

```
[Gmail / External Mail Client]
        │
        ▼
      SMTP (Port 25/465/587)
        │
        ▼
  Postfix (Mail Server)
        │
        ▼
  Local Mailbox (/var/mail/adminuser)
        │
        ▼
  Access via IMAP/POP3 or CLI
```

---

## License

This project is **open-source**. Feel free to use, modify, and learn from it.
