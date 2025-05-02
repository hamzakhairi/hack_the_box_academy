
# 📝 **SMB (Server Message Block) Overview**

This document provides an overview of SMB (Server Message Block) protocol, its configuration, management, and troubleshooting, specifically focusing on Samba, a popular open-source SMB server for Linux/Unix systems.

## 📚 **Table of Contents**

1. [SMB Protocol Summary](https://chatgpt.com/c/68154776-b40c-8003-b26f-282165f2d027#smb-protocol-summary)
    
2. [What is `smb.conf`?](#what-is-smbconf)
    
3. [Default Configuration Breakdown](#default-configuration-breakdown)
    
4. [Printer Shares](#printer-shares)
    
5. [Important Share Settings](#important-share-settings)
    
6. [Dangerous Settings](#dangerous-settings)
    
7. [Restarting the Samba Service](#restarting-the-samba-service)
    
8. [Listing Available SMB Shares with `smbclient`](#listing-available-smb-shares-with-smbclient)
    
9. [Connecting to a Share](#connecting-to-a-share)
    
10. [Useful `smbclient` Commands](#useful-smbclient-commands)
    
11. [Check Samba Connections with `smbstatus`](#check-samba-connections-with-smbstatus)
    
12. [Advanced Footprinting Tools](#advanced-footprinting-tools)
    

---

## 🧩 **SMB Protocol Summary**

**SMB** is a **client-server communication protocol** used mainly in Windows environments for **file and printer sharing**, **network browsing**, and **remote service access**.

### 🧱 Core Concepts:

- **Protocol**: Works over **TCP** (port **445** for modern versions, **137-139** for legacy).
    
- **Access Control**: Managed with **Access Control Lists (ACLs)**.
    
- **Client-Server Model**: The SMB client requests resources, and the server provides them if allowed.
    

### 🔐 SMB Versions:

|Version|Windows Version|Features|
|---|---|---|
|**CIFS/SMB 1**|NT 4.0 / 2000|NetBIOS, TCP support|
|**SMB 2.0**|Vista / Server 2008|Better performance, caching|
|**SMB 2.1**|Windows 7 / Server 2008 R2|Locking mechanisms|
|**SMB 3.0+**|Windows 8+ / Server 2012+|Encryption, multichannel support|
|**SMB 3.1.1**|Windows 10 / Server 2016|AES-128 encryption, integrity check|

### 🐧 **Samba**:

- An open-source SMB server for Linux/Unix.
    
- Implements **CIFS** to enable cross-platform compatibility with Windows.
    

---

## 🔧 **What is `smb.conf`?**

The **`smb.conf`** file is the primary configuration file for Samba, used to configure file and printer shares between Linux and Windows systems.

- **Location**: `/etc/samba/smb.conf`
    
- **Command to View without Comments**:
    

```bash
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

---

## 📝 **Default Configuration Breakdown**

### `[global]` Section

This section contains global settings that apply to all shares unless overridden.

|Setting|Description|
|---|---|
|`workgroup = DEV.INFREIGHT.HTB`|Name of the Windows workgroup or domain|
|`server string = DEVSMB`|Description of the server|
|`log file = /var/log/samba/log.%m`|Log file location per machine|
|`max log size = 1000`|Max size of log files in KB|
|`server role = standalone server`|Type of server (standalone, domain member)|
|`unix password sync = yes`|Sync system password with Samba password|
|`map to guest = bad user`|Map unknown users to guest access|

---

## 🖨️ **Printer Shares**

### Configuration Example:

```ini
[printers]
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
```

|Setting|Description|
|---|---|
|`[printers]`|Share name for printers|
|`path`|Path where print jobs are stored|
|`printable = yes`|Marks the share as a print-only share|
|`guest ok = no`|Denies guest access|
|`read only = yes`|Makes the share read-only|

---

## ⚙️ **Important Share Settings**

|Setting|Description|
|---|---|
|`[sharename]`|Name of the share|
|`path = /path/here/`|Directory being shared|
|`browseable = yes`|Make the share visible in network browsing|
|`guest ok = yes`|Allow guest access without authentication|
|`read only = yes`|Share is read-only|
|`create mask = 0700`|Permissions for new files|
|`writable = yes`|Allow writing to the share|

---

## ⚠️ **Dangerous Settings**

|Setting|Risk|
|---|---|
|`browseable = yes`|Allows attackers to list folders|
|`guest ok = yes`|Allows anyone to connect without authentication|
|`read only = no` / `writable = yes`|Allows unauthorized modifications or deletions of files|
|`create mask = 0777`|New files are created with full permissions|
|`logon script` / `magic script`|Can execute harmful code if misconfigured|

---

## 🔄 **Restarting the Samba Service**

To apply configuration changes or restart the Samba service:

```bash
sudo systemctl restart smbd
```

---

## 📡 **Listing Available SMB Shares with `smbclient`**

To list all available shares on the Samba server:

```bash
smbclient -N -L //10.129.14.128
```

### Explanation:

- `-N`: No password prompt (anonymous login).
    
- `-L`: List available shares.
    
- `//10.129.14.128`: Target server IP.
    

---

## 📥 **Connecting to a Share**

Connect to a specific share (e.g., `notes`) anonymously:

```bash
smbclient //10.129.14.128/notes
```

---

## 📚 **Useful `smbclient` Commands**

At the `smb: \>` prompt:

- `help`: Show available commands
    
- `ls`: List files in the share
    
- `get <filename>`: Download a file
    
- `put <filename>`: Upload a file
    
- `!ls`: Run a local shell command
    
- `!cat <filename>`: Display contents of a local file
    

---

## 👮 **Check Samba Connections with `smbstatus`**

To check who is connected to the Samba server:

```bash
smbstatus
```

This will show:

- Who is connected
    
- From where
    
- To which share
    
- Protocol version
    

---

## ⚙️ **Advanced Footprinting Tools**

### 🛠️ **Nmap**

Nmap is essential for scanning and service detection. Use this command to scan for SMB:

```bash
nmap -p 139,445 --script=smb-enum-shares,smb-enum-users <target-ip>
```

### 📞 **RPCclient**

Use `rpcclient` for Windows RPC enumeration:

```bash
rpcclient -U "" <target-ip>
```

### 🧰 **Impacket**

Impacket tools like `samrdump.py` are useful for extracting domain information:

```bash
python3 samrdump.py <target-ip>
```

### 📂 **SMBmap**

SMBMap helps in auditing SMB shares:

```bash
smbmap -H <target-ip>
```

### 📊 **Enum4linux-ng**

Use `enum4linux-ng` for detailed enumeration of Windows-based SMB shares:

```bash
enum4linux-ng <target-ip>
```

---

## 🧠 **Tips**

- Always begin with anonymous enumeration to gather initial information.
    
- Use `Nmap` to verify open SMB ports before deep enumeration.
    
- Combine outputs from various tools for comprehensive network analysis.
    

---

## 📍 **Lab Target**

Target: `10.129.202.5`

### Nmap Scan:

```bash
nmap -sC -sV -Pn 10.129.202.5
```

#nmapscan 
 
  ```shell
┌──(root㉿kali)-[/home/hamza]
└─# nmap -sC -sV -Pn 10.129.202.5 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-02 21:32 +01
Nmap scan report for 10.129.202.5
...
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
...
|_  100227  3           2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
2049/tcp open  nfs         3-4 (RPC #100003)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: DEVSMB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: -1s
| smb2-time: 
|   date: 2025-05-02T20:35:02
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 222.15 seconds
```

#use_smbclient

- Show accessible share on the target
```shell
┌──(root㉿kali)-[/home/hamza]
└─# smbclient -N -L //10.129.202.5

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sambashare      Disk      InFreight SMB v3.1
	IPC$            IPC       IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
Protocol negotiation to server 10.129.202.5 (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available
```

- Connect to the discovered share
```shell
┌──(root㉿kali)-[/home/hamza]
└─# smbclient //10.129.202.5/sambashare
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Nov  8 14:43:14 2021
  ..                                  D        0  Mon Nov  8 16:53:19 2021
  .profile                            H      807  Tue Feb 25 13:03:22 2020
  contents                            D        0  Mon Nov  8 14:43:45 2021
  .bash_logout                        H      220  Tue Feb 25 13:03:22 2020
  .bashrc                             H     3771  Tue Feb 25 13:03:22 2020

		4062912 blocks of size 1024. 506172 blocks available
smb: \> cd contents
smb: \contents\> dir
  .                                   D        0  Mon Nov  8 14:43:45 2021
  ..                                  D        0  Mon Nov  8 14:43:14 2021
  flag.txt                            N       38  Mon Nov  8 14:43:45 2021

		4062912 blocks of size 1024. 506172 blocks available
smb: \contents\> !cat flag.txt
HTB{-------------------------------------------}
smb: \contents\> 
```

- Find out which Domain the server belongs to :  `DEVOPS`
```shell
┌──(root㉿kali)-[/home/hamza]
└─# rpcclient -U "" 10.129.202.5
Password for [WORKGROUP\]:
rpcclient $> enumdomains
name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]
rpcclient $> enumdomgroups
rpcclient $> querydominfo
Domain:		DEVOPS
Server:		DEVSMB
Comment:	InlaneFreight SMB server (Samba, Ubuntu)
Total Users:	0
Total Groups:	0
Total Aliases:	0
Sequence No:	1746223445
Force Logoff:	4294967295
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
```

- Find additional information about the specific share we found previously : `InFreight SMB v3.1`

```shell
┌──(root㉿kali)-[/home/hamza]
└─# rpcclient -U "" 10.129.202.5
Password for [WORKGROUP\]:
rpcclient $> netshareenumall
netname: print$
	remark:	Printer Drivers
	path:	C:\var\lib\samba\printers
	password:	
netname: sambashare
	remark:	InFreight SMB v3.1
	path:	C:\home\sambauser\
	password:	
netname: IPC$
	remark:	IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
	path:	C:\tmp
	password:	
rpcclient $> 
```

- the full system path of that specific share
```shell
path:	C:\home\sambauser\
```
