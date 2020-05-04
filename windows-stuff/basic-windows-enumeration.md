# Basic Windows Enumeration



```text
nmap -sC -sV $ip -Pn > nmap
```

```text
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='cascade.local' 10.10.10.18
```

```text
ldapsearch -h $ip -x -s base namingcontexts
```

```text
ldapsearch -h $ip -x -b "DC=$DOMAIN,DC=COM" '(ObjectClass=Person)' sAMAccountName | grep sAMAccountName  | awk {'print $2'} > ldap.users
```

```text
smbclient -L //$ip
```

```text
for i in $(cat ldap.users); do smbclient -L $ip -U=$i; done
```

```text
for i in $(cat ldap.users); do smbclient -L $ip -U=$i%$i; done
```

```text
crackmapexec smb $ip --pass-pol -u '' -p ''
```

```text
GetNPUsers.py $domain/ -request -dc-ip $ip -no-pass -usersfile users.txt
```

