# Enumeration

Gobuster - HTTP/s diretory enumeration

`gobuster dir -u http://sneakycorp.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -o gobuster-out.txt`

Fuff - subdomain enumeeration

`ffuf -c -w /usr/share/wordlists/dirb/common.txt -u http://sneakycorp.htb -H "Host: FUZZ.sneakycorp.htb" -fs 185`

Wfuzz - creds, files, domains, fuzzing:

`wfuzz -z file,users -z file,passwords -m zip -u` [`http://10.10.40.100/manage.php`](http://10.10.40.100/manage.php) `-d "username=FUZZ&password=FUZ2Z"`

