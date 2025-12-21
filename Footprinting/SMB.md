## WHAT is SMB ?

`Server Message Block` (`SMB`) is a client-server protocol that regulates access to files and entire directories and other network resources such as printers, routers, or interfaces released for the network. Information exchange between different system processes can also be handled based on the SMB protocol.

## Default Configuration

```bash
hkhairi42@htb[/htb]$ cat /etc/samba/smb.conf | grep -v "#\|\;" 
```

| **Setting**                    | **Description**                                                       |
| ------------------------------ | --------------------------------------------------------------------- |
| `[sharename]`                  | The name of the network share.                                        |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup that will appear when clients query.                        |
| `path = /path/here/`           | The directory to which user is to be given access.                    |
| `server string = STRING`       | The string that will show up when a connection is initiated.          |
| `unix password sync = yes`     | Synchronize the UNIX password with the SMB password?                  |
| `usershare allow guests = yes` | Allow non-authenticated users to access defined share?                |
| `map to guest = bad user`      | What to do when a user login request doesn't match a valid UNIX user? |
| `browseable = yes`             | Should this share be shown in the list of available shares?           |
| `guest ok = yes`               | Allow connecting to the service without using a password?             |
| `read only = yes`              | Allow users to read files only?                                       |
| `create mask = 0700`           | What permissions need to be set for newly created files?              |

|**Setting**|**Description**|
|---|---|
|`browseable = yes`|Allow listing available shares in the current share?|
|`read only = no`|Forbid the creation and modification of files?|
|`writable = yes`|Allow users to create and modify files?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`enable privileges = yes`|Honor privileges assigned to specific SID?|
|`create mask = 0777`|What permissions must be assigned to the newly created files?|
|`directory mask = 0777`|What permissions must be assigned to the newly created directories?|
|`logon script = script.sh`|What script needs to be executed on the user's login?|
|`magic script = script.sh`|Which script should be executed when the script gets closed?|
|`magic output = script.out`|Where the output of the magic script needs to be stored?|

#### Example Share

```shell-session
...SNIP...

[notes]
	comment = CheckIT
	path = /mnt/notes/

	browseable = yes
	read only = no
	writable = yes
	guest ok = yes

	enable privileges = yes
	create mask = 0777
	directory mask = 0777
```

#### SMBclient - Connecting to the Share

```bash
hkhairi42@htb[/htb]$ smbcleint -N -L //IP
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

### 🔹 `-L` → **List shares**
- Lists all available **SMB shares** on the target
- Does **not** connect to a share yet
### 🔹 `-N` → **No password**
- Try **anonymous / null session**
- No password prompt
- Uses empty credentials

```bash
hkhairi42@htb[/htb]$ smbclient //IP/notes
```
Once we have discovered interesting files or folders, we can download them using the `get` command. Smbclient also allows us to execute local system commands using an exclamation mark at the beginning (`!<cmd>`) without interrupting the connection.

#### Download or Read Files from SMB

```bash
smb: \> get prep-prod.txt 
smb: \> !cat prep-prod.txt
```

#### Nmap

```shell-session
hkhairi42@htb[/htb]$ sudo nmap -sS -sC IP -p139,445
```

#### RPCclient

```shell-session
hkhairi42@htb[/htb]$ rpcclient -U "" IP
```

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |
#### SMBmap

```bash
hkhairi42@htb[/htb]$ smbmap -H 10.129.14.128

[+] Finding open SMB ports....
[+] User SMB session established on 10.129.14.128...
[+] IP: 10.129.14.128:445       Name: 10.129.14.128                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        home                                                    NO ACCESS       INFREIGHT Samba
        dev                                                     NO ACCESS       DEVenv
        notes                                                   NO ACCESS       CheckIT
        IPC$                                                    NO ACCESS       IPC Service (DEVSM)
```

#### CrackMapExec

```bash
hkhairi42@htb[/htb]$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

#### Enum4Linux-ng - Installation

```bash
hkhairi42@htb[/htb]$ git clone https://github.com/cddmp/enum4linux-ng.git
hkhairi42@htb[/htb]$ cd enum4linux-ng
hkhairi42@htb[/htb]$ pip3 install -r requirements.txt
```

```shell
hkhairi42@htb[/htb]$ ./enum4linux-ng.py 10.129.14.128 -A
```


## `Questions`

` What version of the SMB server is running on the target system? Submit the entire banner as the answer ?`

-> `Samba smbd 4.6.2`

```bash
┌─[root@htb-gnqoy2cmin]─[/home/htb-ac-1515265]
└──╼ #nmap -sS -sC -sV 10.129.202.5 -p 139,445
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-15 05:48 CST
Nmap scan report for 10.129.202.5
Host is up (0.043s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Host script results:
| smb2-time: 
|   date: 2025-12-15T11:48:13
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: DEVSMB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.10 seconds
```

`What is the name of the accessible share on the target?`

`sambashare`

```bash
┌─[root@htb-gnqoy2cmin]─[/home/htb-ac-1515265]
└──╼ #smbclient -N -L //10.129.202.5

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sambashare      Disk      InFreight SMB v3.1
	IPC$            IPC       IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
protocol negotiation failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

┌─[root@htb-gnqoy2cmin]─[/home/htb-ac-1515265]
└──╼ #smbclient  //10.129.202.5/sambashare
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Nov  8 07:43:14 2021
  ..                                  D        0  Mon Nov  8 09:53:19 2021
  .profile                            H      807  Tue Feb 25 06:03:22 2020
  contents                            D        0  Mon Nov  8 07:43:45 2021
  .bash_logout                        H      220  Tue Feb 25 06:03:22 2020
  .bashrc                             H     3771  Tue Feb 25 06:03:22 2020

		4062912 blocks of size 1024. 506268 blocks available
smb: \>
```

`Connect to the discovered share and find the flag.txt file. Submit the contents as the answer ?`
```bash
smb: \> !ls -la | grep flag.txt
-rw-r--r--  1 htb-ac-1515265 htb-ac-1515265      39 Dec 15 05:00 flag.txt
smb: \> !cat flag.txt
....
```


`Find out which domain the server belongs to ?`

`DEVOPS`

```bash
rpcclient $> querydominfo
Domain:		DEVOPS
```

` Find additional information about the specific share we found previously and submit the customized version of that specific share as the answer ?`

`InFreight SMB v3.1`

```bash
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

```
rpcclient $> netsharegetinfo sambashare
netname: sambashare
	remark:	InFreight SMB v3.1
	path:	C:\home\sambauser\
```

`What is the full system path of that specific share? (format: "/directory/names") ?`

`/home/sambauser`

