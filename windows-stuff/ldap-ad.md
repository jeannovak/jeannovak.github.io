# LDAP / AD

 Default md5sum 

```text
ldapsearch -h 10.10.10.161 
```

Simple authentication 

```text
ldapsearch -h 10.10.10.161 -x
```

A suffix \(also known as a naming context\) is a DN that identifies the top entry in a locally held directory hierarchy.

```text
ldapsearch -h 10.10.10.161 -x -s base namingcontexts

dn: namingContexts: DC=htb,DC=local namingContexts: CN=Configuration,DC=htb,DC=local 
namingContexts: CN=Schema,CN=Configuration,DC=htb,DC=local 
namingContexts: DC=DomainDnsZones,DC=htb,DC=local 
namingContexts: DC=ForestDnsZones,DC=htb,DC=local
```

Get everything from ldap htb.local 

```text
ldapsearch -h 10.10.10.161 -x -b "DC=htb,DC=Dlocal" > ldap-anon.out 
```

Information after -b is a query 

```text
ldapsearch -h 10.10.10.161 -x -b "DC=htb,DC=Dlocal" '(ObjectClass=user)' 
```

Filter what you want from that query 

```text
ldapsearch -h 10.10.10.161 -x -b "DC=htb,DC=Dlocal" '(ObjectClass=user)' sAMAccountName 
ldapsearch -h 10.10.10.161 -x -b "DC=htb,DC=Dlocal" '(ObjectClass=user)' sAMAccountName sAMAccountType 
```

Grep all users

```text
ldapsearch -h 10.10.10.161 -x -b "DC=htb,DC=local" '(ObjectClass=Person)' sAMAccountName | grep sAMAccountName | awk {'print $2'} > userlist.ldap 
```

Check deleted users 

```text
Get-ADObject -IncludeDeletedObjects -Filter {ObjectClass -eq 'user' -and Isdeleted -eq $True} 
```

Filter by displayname and then restore the object

```text
Get-ADObject -IncludeDeletedObjects -Filter {ObjectClass -eq 'user' -and DisplayName -eq 'TempAdmin'} | RestoreADObject 
```

Check details of the user

```text
 Get-ADObject -IncludeDeletedObjects -Filter {ObjectClass -eq 'user' -and DisplayName -eq 'TempAdmin'} -Properties *
```

