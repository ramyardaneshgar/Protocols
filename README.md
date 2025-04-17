# Protocols
Exploitation of insecure protocols using Telnet, FTP, IMAP, and SSH to demonstrate sniffing, MITM, and password attacks, with mitigation via TLS and secure authentication.

By Ramyar Daneshgar 

I explored how legacy protocols—like Telnet, FTP, POP3, and SMTP—can expose sensitive information when encryption is not implemented. My objective was to understand three key types of attacks: sniffing, Man-in-the-Middle (MITM), and password attacks. I also examined how to mitigate these threats using cryptographic protocols such as TLS and SSH. I interacted with a variety of services using Telnet, Netcat, SSH, Tcpdump, Wireshark, and Hydra, gaining hands-on experience in analyzing communication at the packet level and automating attacks.

---

## Sniffing Attack

Sniffing refers to capturing packets that traverse a network. If the protocol is unencrypted (cleartext), the contents—including usernames and passwords—can be directly observed.

**What I Did**:  
I used `tcpdump` to capture POP3 traffic on port 110.

```bash
sudo tcpdump port 110 -A
```

- `sudo`: Required because capturing packets on a network interface requires elevated privileges.
- `port 110`: Filters traffic to only show POP3 communications.
- `-A`: Displays packet contents in ASCII format, useful when looking for credentials.

**Result**:  
I successfully captured:
- `USER frank`
- `PASS D2xc9CgD`

This validated that credentials were sent in plaintext over the network.

**Lessons Learned**:
- Protocols that transmit credentials in plaintext are critically vulnerable.
- Tools like `tcpdump`, when used with root access and specific port filters, can reveal sensitive authentication data.
- To mitigate, protocols like POP3 should always be paired with TLS (POP3S on port 995).

---


What do you need to add to the command `sudo tcpdump` to capture only Telnet traffic?  
**Answer**: `port 23`

What is the simplest display filter you can use with Wireshark to show only IMAP traffic?  
**Answer**: `imap`

---

##  Man-in-the-Middle (MITM) Attack

**Concept**: MITM occurs when an attacker intercepts and potentially alters traffic between two parties who believe they are communicating directly. It targets the *integrity* of the communication.

**Tools Used**:  
- **Ettercap**: Offers 3 interfaces (text, graphical, and ncurses-based).
- **Bettercap**: A powerful alternative with support for scriptable MITM attacks.

**Answering Questions**:

How many different interfaces does Ettercap offer?  
**Answer**: 3

In how many ways can you invoke Bettercap?  
**Answer**: 3

**Lessons Learned**:
- Cleartext protocols like HTTP, FTP, and POP3 are vulnerable to manipulation if no authentication or cryptographic signing is used.
- TLS is effective in preventing MITM by ensuring mutual authentication and message integrity through the use of certificates and encrypted channels.

---

## Encryption with TLS/SSL

**Concept**: TLS (Transport Layer Security) ensures confidentiality and integrity at the presentation layer of the OSI model. It replaces SSL and is used to encrypt protocols like HTTP (becomes HTTPS), SMTP, POP3, and FTP.

**Steps to Establish TLS**:
1. Client sends `ClientHello`.
2. Server responds with `ServerHello`, provides its certificate.
3. Client validates the certificate and sends a pre-master key.
4. Both switch to encrypted communication using a shared session key.

Even if an attacker intercepts traffic, without access to the negotiated key, they cannot decrypt the contents.

**4.1** DNS can also be secured using TLS. What is the three-letter acronym of the DNS protocol that uses TLS?  
**Answer**: DoT (DNS over TLS)

**Lessons Learned**:
- TLS is essential for upgrading insecure protocols.
- The presence of a certificate alone isn't enough; the client must validate it against a trusted certificate authority.
- Applications relying on TLS should monitor for expiry, revocation, or weak cipher suites.

---

## SSH and SCP

**Concept**: SSH provides a secure remote shell with encryption and authentication, typically over port 22. It supports both password-based and key-based authentication. SCP is a file transfer mechanism that runs over SSH.

**Command Used**:

```bash
ssh mark@MACHINE_IP
```

Then I authenticated using the password `XBtc49AB`.

To determine the kernel version:

```bash
uname -r
```

**Answer (5.1)**:  
Originally: `5.4.0-84-generic`  
Updated (based on recent runs): `5.15.0-119-generic`

To transfer a file using SCP:

```bash
scp mark@MACHINE_IP:/home/mark/book.txt .
```

**Answer (5.2)**:  
Download size shown by `scp`: `415 KB`

**Lessons Learned**:
- SSH eliminates the need for Telnet by encrypting session data and file transfers.
- Always verify host fingerprints when connecting for the first time to prevent MITM.
- SCP offers a secure and simple way to transfer files over the same encrypted channel as SSH.

---

## Task 6: Password Attacks with Hydra

**Concept**: Many services require user authentication. If weak passwords are used, they can be cracked using dictionary attacks or brute-force attacks with tools like THC Hydra.

**Tool Used**: Hydra  
**Command Syntax**:

```bash
hydra -l lazie -P /usr/share/wordlists/rockyou.txt MACHINE_IP imap
```

- `-l lazie`: Target username.
- `-P rockyou.txt`: Common password list from the 2009 RockYou breach.
- `imap`: The target protocol.

**Result**:  
Hydra discovered that the password for the user `lazie` on IMAP was `butterfly`.

**Answer (6.1)**: `butterfly`

**Lessons Learned**:
- Password attacks are highly effective when the password policy is weak.
- Dictionary attacks are much faster than brute force, but depend on leaked or common passwords.
- Account lockouts and 2FA can be very effective in mitigating these attacks.

---

## Summary and Lessons Learned

I explored how sniffing, MITM, and password attacks exploit insecure protocols and misconfigurations. For each:

- **Sniffing** targets confidentiality. Preventable with TLS.
- **MITM** targets integrity. Preventable with mutual authentication and encryption.
- **Password attacks** target weak credentials. Preventable with strong password policies and multi-factor authentication.


| Protocol | Port | Secure Version | Secure Port |
|----------|------|----------------|-------------|
| FTP      | 21   | FTPS            | 990         |
| HTTP     | 80   | HTTPS           | 443         |
| POP3     | 110  | POP3S           | 995         |
| IMAP     | 143  | IMAPS           | 993         |
| SMTP     | 25   | SMTPS           | 465         |
| Telnet   | 23   | SSH             | 22          |
| SFTP     | —    | SSH-based       | 22          |

