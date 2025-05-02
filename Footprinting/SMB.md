Here’s a **general summary (résumé) of the SMB module** from HTB Academy, written in a clear and concise way, ideal for adding to your GitHub README or repo description:

---

## 📁 SMB (Server Message Block) – Summary

**SMB** is a **client-server communication protocol** used mainly in Windows environments for **file and printer sharing**, **network browsing**, and **remote service access**. It allows systems to access shared resources like files and directories over a network.

### 🧱 Core Concepts:
- **Protocol**: Works over **TCP**, mainly using port **445** (modern) and **137–139** (legacy via NetBIOS).
- **Access Control**: Permissions are managed using **Access Control Lists (ACLs)**, which control read, write, and execute access.
- **Client-Server Model**: The SMB client requests resources, and the server shares them if permissions allow.
### 🐧 Samba:
- An open-source **SMB server for Linux/Unix**.
- Implements **CIFS** (a dialect of SMB), enabling **cross-platform compatibility** with Windows systems.
- Samba enables Linux machines to **join Windows domains** and act as **domain controllers**.

### 🔐 SMB Versions:

|Version|Windows Version|Features|
|---|---|---|
|**CIFS/SMB 1**|NT 4.0 / 2000|NetBIOS, TCP support|
|**SMB 2.0**|Vista / Server 2008|Better performance, caching|
|**SMB 2.1**|Windows 7 / Server 2008 R2|Locking mechanisms|
|**SMB 3.0+**|Windows 8+ / Server 2012+|Encryption, multichannel, integrity|
|**SMB 3.1.1**|Windows 10 / Server 2016|AES-128 encryption, integrity check|

### 🧠 Important Tools and Terms:

- **NetBIOS**: Legacy naming and session service API.
- **WINS (Windows Internet Name Service)**: Enhanced name resolution.
- **Workgroup**: Logical group of computers in SMB networks.

---
## 🔧 What is `smb.conf`?

`smb.conf` is the **main configuration file** for the **Samba** service, which is used to share directories and printers between Linux and Windows systems over a network.

It’s usually located at:

```
/etc/samba/smb.conf
```

You can view it without comments (lines starting with `#` or `;`) using:

```bash
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

---

## ✅ Default Configuration Breakdown

When you run the command above, you get something like:

```ini
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   ...
```

### 🔹 `[global]` Section

This section contains **global settings** that apply to all shares unless specifically overridden.

#### Key Settings:

|Setting|Description|
|---|---|
|`workgroup = DEV.INFREIGHT.HTB`|Name of the Windows workgroup or domain|
|`server string = DEVSMB`|Description that shows up when clients connect|
|`log file = /var/log/samba/log.%m`|Location of Samba log file per machine|
|`max log size = 1000`|Max size of log files in KB|
|`server role = standalone server`|Type of Samba server (e.g., standalone, domain member)|
|`unix password sync = yes`|Sync Samba password with system password|
|`map to guest = bad user`|Map unknown users to guest access|

---

## 🖨️ Printer Shares

```ini
[printers]
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
```

This section shares printers. Let’s break it down:

|Setting|Meaning|
|---|---|
|`[printers]`|Share name for printers|
|`path`|Where print jobs are stored|
|`printable = yes`|Makes it a print-only share|
|`guest ok = no`|Guests can't access it|
|`read only = yes`|Files can't be modified|

---

## 🧾 Important Share Settings

|Setting|Description|
|---|---|
|`[sharename]`|Name of the share|
|`path = /path/here/`|Directory being shared|
|`browseable = yes`|Show share in network listing|
|`guest ok = yes`|Allow guest (unauthenticated) access|
|`read only = yes`|Share is read-only|
|`create mask = 0700`|Permissions for new files|
|`writable = yes`|Allow file editing and creation|
|`enable privileges = yes`|Use per-user Windows-style privileges|
|`directory mask = 0777`|Permissions for new directories|

---

## ⚠️ Dangerous Settings

These settings can be risky if misconfigured:

|Setting|Risk|
|---|---|
|`browseable = yes`|Attacker can list folders|
|`guest ok = yes`|Anyone can connect without a password|
|`read only = no` / `writable = yes`|Files can be changed or deleted|
|`create mask = 0777`|New files get full permissions|
|`logon script` / `magic script`|Can run code automatically, which could be abused|

---

## 🔄 Restarting the Samba Service

To apply any changes in the Samba configuration or to simply make sure the service is running, use:

```bash
sudo systemctl restart smbd
```

---

## 📡 Listing Available SMB Shares with `smbclient`

From your machine, use `smbclient` to **list all available shares** on the Samba server:

```bash
smbclient -N -L //10.129.14.128
```

### Explanation:

- `-N`: Tells the client to **not prompt for a password** (anonymous login).
    
- `-L`: Lists the shares on the remote server.
    
- `//10.129.14.128`: IP address of the Samba server.
    

### Output Example:

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
home            Disk      INFREIGHT Samba
dev             Disk      DEVenv
notes           Disk      CheckIT
IPC$            IPC       IPC Service (DEVSM)
```

You now know the names of the available shares. The ones we care about are usually custom ones like `notes`, `dev`, etc. Default ones like `print$` and `IPC$` are part of the system.

---

## 📥 Connecting to a Share

Now connect to a specific share, such as `notes`, anonymously:

```bash
smbclient //10.129.14.128/notes
```

Then press Enter when it prompts for a password.

You’ll see:

```
Anonymous login successful
Try "help" to get a list of possible commands.
```

---

## 📚 Useful `smbclient` Commands

At the prompt (`smb: \>`), you can:

- `help`: Show all available commands.
    
- `ls`: List files in the share.
    
- `get <filename>`: Download a file.
    
- `put <filename>`: Upload a file.
    
- `!ls`: Run a **local shell command**.
    
- `!cat <filename>`: Display contents of a local file.
    

### Example:

```bash
smb: \> ls
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021
```

Download it:

```bash
smb: \> get prep-prod.txt
```

Then list files **locally** to confirm it was saved:

```bash
smb: \> !ls
```

And view the contents:

```bash
smb: \> !cat prep-prod.txt
```

---

## 👮 Check Samba Connections with `smbstatus`

On the **Samba server**, you can check who is connected:

```bash
smbstatus
```

Example output:

```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine        Protocol Version
-----------------------------------------------------------------
75691   sambauser    samba        10.10.14.4     SMB3_11

Service  pid     Machine       Connected at
--------------------------------------------
notes    75691   10.10.14.4    Sep 23 00:12:06
```

This shows:

- Who is connected
    
- From where
    
- To which share
    
- Protocol version used
    

---

## 🛡️ Extra: Domain Mode (Advanced)

In more advanced environments, Samba can work in a **Windows Domain**, where:

- A **domain controller** handles user authentication (usually a Windows server).
    
- Samba connects to the domain and relies on it to verify users.
    
- The domain controller manages users/passwords via `NTDS.dit` and `SAM`.
    

This setup is useful for enterprise networks with centralized user management.

---

## 🔍 Footprinting with Nmap, RPCclient, samrdump.py, SMBmap, and Enum4linux-ng

Footprinting is the first step in the information gathering phase of ethical hacking and penetration testing. The goal is to collect as much information as possible about a target system. Here are powerful tools used for footprinting, especially for SMB and Windows-based environments:

---

### 📡 Nmap – Scanning and Service Detection

**Nmap (Network Mapper)** is a fundamental tool for footprinting and port scanning.

#### Useful Commands:

```bash
nmap -sC -sV -Pn -p- <target-ip>
```

- `-sC`: Runs default scripts (good for enumeration)
- `-sV`: Version detection
- `-Pn`: Skip host discovery
- `-p-`: Scan all 65535 ports

To specifically target SMB ports:

```bash
nmap -p 139,445 --script=smb-enum-shares,smb-enum-users <target-ip>
```

---

### 📞 RPCclient – Windows RPC Enumeration

`rpcclient` is part of the **Samba suite** and is used to interact with Windows RPC endpoints.

#### Connect anonymously:

```bash
rpcclient -U "" <target-ip>
```

Then you can try:

- `enumdomusers` → Enumerate domain users
    
- `queryuser <RID>` → Query info about a specific user
    
- `enumdomgroups` → List domain groups
    
- `lookupsids` → Convert SIDs to names
    

---

### 🧰 Impacket - samrdump.py

**Impacket** is a powerful collection of Python scripts for Windows protocols.

#### Samrdump.py usage:

```bash
python3 samrdump.py <ip>
```

It connects to the **SAMR (Security Account Manager Remote)** protocol and dumps:

- Domain users
    
- Domain groups
    
- RIDs
    
- SID information
    

Example:

```bash
python3 /opt/impacket/examples/samrdump.py <target-ip>
```

---

### 📂 SMBmap – SMB Share Enumeration

SMBMap helps in auditing SMB shares on a network.

#### Basic usage:

```bash
smbmap -H <target-ip>
```

To try with credentials:

```bash
smbmap -u <user> -p <pass> -H <target-ip>
```

It shows:

- Shared folders
    
- Access level (READ, WRITE, NO ACCESS)
    
- Paths
    

You can also download files using `-R <share>` or upload files.

---

### 📊 Enum4linux-ng – Advanced SMB Enumeration

**Enum4linux-ng** is a rewrite of the classic `enum4linux` script, focused on Windows enumeration via SMB.

#### Basic anonymous scan:

```bash
enum4linux-ng <target-ip>
```

You can also specify credentials:

```bash
enum4linux-ng -u <user> -p <pass> <target-ip>
```

It gathers:

- NetBIOS names
- Domain, users, groups
- Password policies
- Shares
- OS version

---

### 🧠 Tips:

- Always try **anonymous enumeration** first before using credentials.
- Use **Nmap scripts** to verify services before deep enumeration.
- Combine output from all tools to create a comprehensive picture of the target.

---

## Lab target  (10.129.202.5)


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
