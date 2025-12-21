## WHAT is FTP ?

The `File Transfer Protocol` (`FTP`) is one of the oldest protocols on the Internet. The FTP runs within the application layer of the TCP/IP protocol stack. Thus, it is on the same layer as `HTTP` or `POP`. These protocols also work with the support of browsers or email clients to perform their services. There are also special FTP programs for the File Transfer Protocol.

## Default Configuration

One of the most used FTP servers on Linux-based distributions is [vsFTPd](https://security.appspot.com/vsftpd.html). The default configuration of vsFTPd can be found in `/etc/vsftpd.conf`, and some settings are already predefined by default. It is highly recommended to install the vsFTPd server on a VM and have a closer look at this configuration.

#### Install vsFTPd

```shell-session
hkhairi42@htb[/htb]$ sudo apt install vsftpd 
```

#### vsFTPd Config File

```shell-session
hkhairi42@htb[/htb]$ cat /etc/vsftpd.conf | grep -v "#"
```

| **Setting**                                                   | **Description**                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `listen=NO`                                                   | Run from inetd or as a standalone daemon?                                                                |
| `listen_ipv6=YES`                                             | Listen on IPv6 ?                                                                                         |
| `anonymous_enable=NO`                                         | Enable Anonymous access?                                                                                 |
| `local_enable=YES`                                            | Allow local users to login?                                                                              |
| `dirmessage_enable=YES`                                       | Display active directory messages when users go into certain directories?                                |
| `use_localtime=YES`                                           | Use local time?                                                                                          |
| `xferlog_enable=YES`                                          | Activate logging of uploads/downloads?                                                                   |
| `connect_from_port_20=YES`                                    | Connect from port 20?                                                                                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Name of an empty directory                                                                               |
| `pam_service_name=vsftpd`                                     | This string is the name of the PAM service vsftpd will use.                                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | The last three options specify the location of the RSA certificate to use for SSL encrypted connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` |                                                                                                          |
| `ssl_enable=NO`                                               |                                                                                                          |
#### FTPUSERS

```shell-session
hkhairi42@htb[/htb]$ cat /etc/ftpusers

guest
john
kevin
```

#### Download All Available Files

```shell-session
hkhairi42@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

#### Upload a File

```shell-session
hkhairi42@htb[/htb]$ touch testupload.txt
```

```shell-session
ftp> put testupload.txt 
```

## Nmap FTP Scripts

```shell-session
hkhairi42@htb[/htb]$ sudo nmap --script-updatedb
```

```shell-session
hkhairi42@htb[/htb]$ find / -type f -name ftp* 2> /dev/null | grep script
```

```
hkhairi42@htb[/htb]$ sudo nmap -sS -sC -sV -p21 IP -A 

	Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
	Nmap scan report for 10.129.14.136
	Host is up (0.00013s latency).
	
	PORT   STATE SERVICE VERSION
	21/tcp open  ftp     vsftpd 2.0.8 or later
	| ftp-anon: Anonymous FTP login allowed (FTP code 230)
	| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
	| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
	| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
	| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
	| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
	|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
	| ftp-syst: 
	|   STAT: 
	| FTP server status:
	|      Connected to 10.10.14.4
	|      Logged in as ftp
	|      TYPE: ASCII
	|      No session bandwidth limit
	|      Session timeout in seconds is 300
	|      Control connection is plain text
	|      Data connections will be plain text
	|      At session startup, client count was 2
	|      vsFTPd 3.0.3 - secure, fast, stable
	|_End of status

```

#### Service Interaction

```shell-session
hkhairi42@htb[/htb]$ nc -nv 10.129.14.136 21
```

### 🔹 `-n` → **No DNS**
- Do **not** resolve IP to hostname
- Faster
- Avoids DNS leaks / delays
### 🔹 `-v` → **Verbose**
- Shows connection info
- Tells you if connection succeeds or fails

```shell-session
hkhairi42@htb[/htb]$ telnet 10.129.14.136 21
```

```shell-session
hkhairi42@htb[/htb]$ openssl s_client -connect IP:PORT -starttls ftp
```


### **Questions**

`` Which version of the FTP server is running on the target system? Submit the entire banner as the answer ?

`InFreight FTP v1.1`

```
┌─[eu-academy-6]─[10.10.15.151]─[htb-ac-1515265@htb-gnqoy2cmin]─[~]
└──╼ [★]$ sudo nmap -sC -sV 10.129.202.5 -p 21
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-15 04:52 CST
Stats: 0:00:12 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for 10.129.202.5
Host is up (0.044s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 ftpuser  ftpuser        39 Nov  8  2021 flag.txt
| fingerprint-strings: 
|   GenericLines: 
|     220 InFreight FTP v1.1
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 InFreight FTP v1.1

```

` Enumerate the FTP server and find the flag.txt file. Submit the contents of it as the answer ?`

after login 
```
ftp> ls
229 Entering Extended Passive Mode (|||15956|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftpuser  ftpuser        39 Nov  8  2021 flag.txt
226 Transfer complete
```

you have **tow methods** 
```
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||33440|)
150 Opening BINARY mode data connection for flag.txt (39 bytes)
    39       61.03 KiB/s 
226 Transfer complete
39 bytes received in 00:00 (0.74 KiB/s)
```
or use `wget` 
```
┌─[eu-academy-6]─[10.10.15.151]─[htb-ac-1515265@htb-gnqoy2cmin]─[~]
└──╼ [★]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.202.5
--2025-12-15 05:03:53--  ftp://anonymous:*password*@10.129.202.5/
           => ‘10.129.202.5/.listing’
Connecting to 10.129.202.5:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.
....
```

