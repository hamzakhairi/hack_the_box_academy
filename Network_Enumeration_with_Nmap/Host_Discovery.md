Host Discovery
==============

**Scan Network Range**

```bash
$ sudo nmap IP/24 -sn -oA tenet 
```

IP/24		: Target network range.
-sn			: Disables port scannig 
-oA tent	: Stores the results in all formats


**Scan IP List**

if you have list like 

```bash 
$cat IP.list
10.129.2.4
10.129.2.10
10.129.2.11
```

and the you do this
```bash
$ sudo nmap -sn -oA tnet -iL IP.list | grep for | cut -d" " -f5
```

`+ 1  Based on the last result, find out which operating system it belongs to. Submit the name of the operating system as result.`

`is : Windows `

becouse the ttl = 128 A TTL value of 128 in a ping response indicates that the packet originated from a device, typically a Windows operating system.


