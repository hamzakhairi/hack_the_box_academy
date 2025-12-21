
## What is SMTP ?

The `Simple Mail Transfer Protocol` (`SMTP`) is a protocol for sending emails in an IP network. It can be used between an email client and an outgoing mail server or between two SMTP servers. SMTP is often combined with the IMAP or POP3 protocols, which can fetch emails and send emails. In principle, it is a client-server-based protocol, although SMTP can be used between a client and a server and between two SMTP servers. In this case, a server effectively acts as a client.

## Footprinting the Service

#### Nmap

```bash
hkhairi42@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

#### Nmap - Open Relay

```bash
hkhairi42@htb[/htb]$ sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

### `Questions`

`Enumerate the SMTP service and submit the banner, including its version as the answer ?`

`InFreight ESMTP v2.11`

```bash
┌─[✗]─[root@htb-en4ebbzoce]─[/home/htb-ac-1515265]
└──╼ #nc -nv 10.129.184.101 25
(UNKNOWN) [10.129.184.101] 25 (smtp) open

220 InFreight ESMTP v2.11
500 5.5.2 Error: bad syntax
```

`Enumerate the SMTP service even further and find the username that exists on the system. Submit it as the answer ?`

