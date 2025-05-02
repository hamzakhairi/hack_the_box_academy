
# Table of Contents

1. [FTP (File Transfer Protocol)](#-ftp-file-transfer-protocol)
    - [What is FTP?](#-what-is-ftp)
    - [How FTP Works](#-how-ftp-works)
    - [Active vs Passive FTP](#-active-vs-passive-ftp)
    - [FTP Commands & Status Codes](#-ftp-commands-status-codes)
    - [Authentication](#-authentication)
2. [TFTP (Trivial File Transfer Protocol)](#-tftp-trivial-file-transfer-protocol)
    - [What is TFTP?](#-what-is-tftp)
    - [How TFTP Works](#-how-tftp-works)
    - [TFTP Commands](#-tftp-commands)
3. [vsFTPd – Very Secure FTP Daemon](#-vsftpd-very-secure-ftp-daemon)
    - [Default Configuration File](#-default-configuration-file)
    - [Installing vsFTPd](#-installing-vsftpd)
    - [Reading the Config File](#-reading-the-config-file)
    - [Dangerous Settings](#-dangerous-settings)
4. [Lab Target (10.129.202.5)](#-lab-target-101292025)

### 🟢 **FTP (File Transfer Protocol)**

#### 🔹 **What is FTP?**

FTP is one of the oldest Internet protocols. It helps users transfer files **between a client and a server**. It works at the **Application Layer** in the TCP/IP model — just like **HTTP** (web) or **POP** (email).

You can use FTP:

- In a **web browser**
    
- With **email clients**
    
- Or better, through special programs made for FTP (like FileZilla)
    

---

#### 🔹 **How FTP Works**

1. **Two connections (channels)** are used:
    
    - **Control Channel**:
        
        - Uses **TCP port 21**
            
        - Used to **send commands** (like upload, delete, etc.) and **receive replies**
            
    - **Data Channel**:
        
        - Uses **TCP port 20**
            
        - Transfers the **actual files**
            
        - Automatically detects errors, and you can resume downloads/uploads if interrupted.
            

---

#### 🔹 **Active vs Passive FTP**

| Mode        | Description                                                                                | Issue                                                         | Solution |
| ----------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------- | -------- |
| **Active**  | Client connects to port 21 and tells the server which client port to use for data.         | Fails if client has a **firewall** that blocks incoming data. | ❌        |
| **Passive** | Server tells the client which port to use. Then the **client starts** the data connection. | Works behind a firewall.                                      | ✅        |

---

#### 🔹 **FTP Commands & Status Codes**

- FTP uses **commands** (like `STOR`, `RETR`, etc.) and replies with **status codes** (like `200 OK`, `530 Not logged in`, etc.).
    
- Not all FTP servers support all commands.
    

---

#### 🔹 **Authentication**

- **Most FTP servers need a username & password.**
    
- But **FTP is insecure** — it sends data in **plain text** (including passwords), which can be **sniffed**.
    
- Some servers allow **anonymous FTP**:
    
    - No password needed
        
    - Usually only **download** is allowed
        
    - Very **limited permissions** due to security concerns
        

---

### 🔵 **TFTP (Trivial File Transfer Protocol)**

#### 🔹 **What is TFTP?**

- A **simpler and lighter** version of FTP
    
- **No authentication**
    
- Uses **UDP** (not TCP) → **less reliable**
    
- Only used in **secure, local networks**
    

---

#### 🔹 **How TFTP Works**

- Transfers files without login
    
- Only works in **shared directories**
    
- No passwords, no user verification
    
- File permissions are controlled by the **OS**, not the protocol
    

---

#### 🔹 **TFTP Commands**

| Command   | Description                                  |
| --------- | -------------------------------------------- |
| `connect` | Set the remote host (and port) to connect to |
| `get`     | Download file(s) from server to client       |
| `put`     | Upload file(s) from client to server         |
| `quit`    | Exit TFTP session                            |
| `status`  | Show current connection info                 |
| `verbose` | Show more details during transfer (on/off)   |
⚠️ **No `ls` or `dir`** — You **cannot list directory contents** like in FTP.

---

### 🟢 **vsFTPd – Very Secure FTP Daemon**

**vsFTPd** is one of the most commonly used and trusted FTP servers on Linux systems. It stands for:

> **Very Secure FTP Daemon**

---

### 🔹 **Default Configuration File**

The config file is located at:

```
/etc/vsftpd.conf
```

This file controls how the FTP server behaves — what features are enabled, who can access it, how security is handled, etc.

---

### 🛠️ **Installing vsFTPd**

You can install it with this command:

```bash
sudo apt install vsftpd
```

- This installs the vsFTPd package.
    
- It's simple and easy to configure compared to more complex FTP servers.
    
- Many configuration options are already there but commented (`#`) — these are disabled by default.
    
- Not all possible settings are shown in this file; to see the full list, you can check the manual:
    

```bash
man vsftpd.conf
```

---

### 🔹 **Reading the Config File**

To view active settings (ignore comments), run:

```bash
cat /etc/vsftpd.conf | grep -v "#"
```

Here’s what the output means:

|Setting|Meaning|
|---|---|
|`listen=NO`|Do not run as a standalone service (controlled by `inetd`)|
|`listen_ipv6=YES`|Listen for incoming IPv6 connections|
|`anonymous_enable=NO`|Don't allow anonymous users (only valid local users can log in)|
|`local_enable=YES`|Allow local Linux system users to log in|
|`dirmessage_enable=YES`|Show messages when users enter certain directories (like `.message` files)|
|`use_localtime=YES`|Use the server's local time (instead of UTC)|
|`xferlog_enable=YES`|Enable logging of uploads and downloads|
|`connect_from_port_20=YES`|Use port 20 for FTP data connections|
|`secure_chroot_dir=/var/run/vsftpd/empty`|A secure jail directory for users — they can’t access system files|
|`pam_service_name=vsftpd`|Specifies the PAM (Pluggable Authentication Module) service used|
|`rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`|Path to the SSL certificate (for encrypted connections)|
|`rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key`|Private key for SSL|
|`ssl_enable=NO`|SSL is disabled by default (you can turn this on for secure FTP using FTPS)|

---

### 🚫 **/etc/ftpusers – User Access Control**

This file lists **users who are denied access to the FTP server**, even if they exist on the system.

For example, if `/etc/ftpusers` contains:

```shell
guest
john
kevin
```

Then these users **cannot log in** to the FTP server — no matter what their password is.

This is used for **security purposes**, to block certain system users from accessing FTP.

---

|Element|Purpose|
|---|---|
|`vsftpd.conf`|Main config file controlling FTP behavior|
|`#` comments|Disabled lines (not active unless uncommented)|
|`grep -v "#" file`|Show only active settings|
|`/etc/ftpusers`|Deny specific users from accessing FTP|
|SSL settings|Present, but off by default|
|Logging|Enabled to keep track of uploads/downloads|

---

## 🔹Dangerous Settings

|**Setting**|**Description**|
|---|---|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

---
#### 🔹Download a File

```shell
ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxrwx    1 ftp      ftp             0 Sep 16 17:24 Calendar.pptx
drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees
-rwxrwxrwx    1 ftp      ftp            41 Sep 18 15:58 Important Notes.txt
226 Directory send OK.


ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)


ftp> exit

221 Goodbye.
```

---
#### 🔹Download All Available Files

```shell
hkhairi42@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/                                         
           => ‘10.129.14.136/.listing’                                                                     
Connecting to 10.129.14.136:21... connected.                                                               
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.                                                                 
12.12.1.136/.listing           [ <=>                                  ]     466  --.-KB/s    in 0s       
                                                                                                         
2021-09-19 14:45:58 (65,8 MB/s) - ‘10.129.14.136/.listing’ saved [466]                                     
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/Calendar.pptx   
           => ‘10.129.14.136/Calendar.pptx’                                       
==> CWD not required.                                                           
==> SIZE Calendar.pptx ... done.                                                                                                                            
==> PORT ... done.    ==> RETR Calendar.pptx ... done.       

...SNIP...

2021-09-19 14:45:58 (48,3 MB/s) - ‘10.129.14.136/Employees/.listing’ saved [119]

FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```

#### 🔹Upload a File

```shell
hkhairi42@htb[/htb]$ touch testupload.txt
```

With the `PUT` command, we can upload files in the current folder to the FTP server.

```shell
ftp> put testupload.txt 

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.
```

---

## Lab  target (10.129.202.5)

🔹Nmap scan 

```shell
┌──(root㉿kali)-[/home/hamza/Desktop]
└─# nmap -sC -p21 -A -sV 10.129.202.5 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-02 18:35 +01
Stats: 0:01:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 87.50% done; ETC: 18:36 (0:00:02 remaining)
Stats: 0:01:16 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 87.50% done; ETC: 18:36 (0:00:04 remaining)
Nmap scan report for 10.129.202.5
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 ftpuser  ftpuser        39 Nov  8  2021 flag.txt
| fingerprint-strings: 
|   GenericLines: 
|     220 InFreight FTP v1.1
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=5/2%Time=6815026C%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,74,"220\x20InFreight\x20FTP\x20v1\.1\r\n500\x20Invalid\x20co
SF:mmand:\x20try\x20being\x20more\x20creative\r\n500\x20Invalid\x20command
SF::\x20try\x20being\x20more\x20creative\r\n");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (95%), Linux 5.0 (95%), Linux 5.0 - 5.4 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), HP P2000 G3 NAS device (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   63.88 ms 10.10.14.1
2   64.02 ms 10.129.202.5

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.66 seconds
```

The Nmap scan shows that the FTP server on `10.129.202.5` allows anonymous login, which can be risky depending on the files accessible (like `flag.txt`). A successful FTP login without authentication indicates a potential vulnerability that could be exploited.

```shell
┌──(root㉿kali)-[/home/hamza/Desktop]
└─# ftp anonymous@10.129.202.5
Connected to 10.129.202.5.
220 InFreight FTP v1.1
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||58322|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftpuser  ftpuser        39 Nov  8  2021 flag.txt
226 Transfer complete
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||54961|)
150 Opening BINARY mode data connection for flag.txt (39 bytes)
    39      103.49 KiB/s 
226 Transfer complete
39 bytes received in 00:00 (0.68 KiB/s)
ftp> exit
221 Goodbye.
```

or if we want download all file we can use this command :

```shell 
wget -m --no-passive ftp://anonymous:anonymous@10.129.202.5
```

finally we get this flag as we need 

THANK YOU FOR READING MY FTP DOCUMENTATION .
