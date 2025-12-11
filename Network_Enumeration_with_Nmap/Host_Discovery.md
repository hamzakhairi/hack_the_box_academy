Host Discovery
==============

# **Scan Network Range**

```bash
$ sudo nmap IP/24 -sn -oA tenet 
```

IP/24		: Target network range.
-sn			: Disables port scannig 
-oA tent	: Stores the results in all formats


# **Scan IP List**

if you have list like 

```bash 
$cat IP.list
10.129.2.4
10.129.2.10
10.129.2.11
```






