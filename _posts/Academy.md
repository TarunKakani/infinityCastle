---
title: "Academy VM"
date: 2026-10-12 20:59 +0530
categories: [Pwned Boxes]
tags: [pwn]
description: Academy Vulnerable VM Exploitation | Easy
---

## Summary

Academy is easy intentionaly vulnerable virtual machine. Made for learning penetration testing at its foundational level.

## Recon

Used Nmap for mapping out the attack surface

```console
$ nmap -A -T4 -p- <ip>

Host is up (0.00024s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:96:ec:08:2f:b1:ea:06:ca:fc:46:8a:7e:8a:e3:55 (RSA)
|   256 56:32:3b:9f:48:2d:e0:7e:1b:df:20:f8:03:60:56:5e (ECDSA)
|_  256 95:dd:20:ee:6f:01:b6:e1:43:2e:3c:f4:38:03:5b:36 (ED25519)
80/tcp    open  http     Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Bolt - Installation error
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      34150/udp   mountd
|   100005  1,2,3      43455/tcp6  mountd
|   100005  1,2,3      46812/udp6  mountd
|   100005  1,2,3      53565/tcp   mountd
|   100021  1,3,4      39923/udp6  nlockmgr
|   100021  1,3,4      43081/tcp6  nlockmgr
|   100021  1,3,4      44117/tcp   nlockmgr
|   100021  1,3,4      60685/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs      3-4 (RPC #100003)
8080/tcp  open  http     Apache httpd 2.4.38 ((Debian))
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: PHP 7.3.27-1~deb10u1 - phpinfo()
44117/tcp open  nlockmgr 1-4 (RPC #100021)
46069/tcp open  mountd   1-3 (RPC #100005)
53565/tcp open  mountd   1-3 (RPC #100005)
55145/tcp open  mountd   1-3 (RPC #100005)
```

- The ports 80 and 8080 were the most interesting. Targeting them was the best low hanging fruit
- Another interesting find was the NFS (network file share) port 2049

After visiting both the webpages hosted at 80 and 8080
- The service hosted was Boltwire
- Searching the exploitDb database for the word Boltwire, there was a LFI vulnerability in Boltwire 6.03

Then before processing further to the exploitation phase, ran a directory bruteforcing scan on both ports

```console
$ gobuster dir -u http://<ip>:80 -w /seclists/Discovery/WebContent/DirBuster-2007_directory-list-2.3-medium.txt -x .php,.txt,.pdf,.html -t 40

index.php            (Status: 200) [Size: 3833]
public               (Status: 301) [Size: 317] [--> http://<ip>/public/]
src                  (Status: 301) [Size: 314] [--> http://<ip>/src/]
app                  (Status: 301) [Size: 314] [--> http://<ip>/app/]
vendor               (Status: 301) [Size: 317] [--> http://<ip>/vendor/]
extensions           (Status: 301) [Size: 321] [--> http://<ip>/extensions/]
server-status        (Status: 403) [Size: 279]
```

```console
gobuster dir -u http://<ip>:8080 -w /seclists/Discovery/WebContent/DirBuster-2007_directory-list-2.3-medium.txt -x .php,.txt,.pdf,.html -t 40

index.php            (Status: 200) [Size: 94532]
dev                  (Status: 301) [Size: 321] [--> http://<ip>:8080/dev/]
server-status        (Status: 403) [Size: 281]
```

After checking all the directories that were exposed, the most juiciest one was /app
- It had a exposed config.yml configuration for the server
- Which gave details for a sqlite database (Worth Noting)

The next thing to enumerate was to try and mount the NFS server onto a local directory in /mnt

```console
$ showmount -t <ip>
$ mkdir /mnt/<dirName>
$ mount -t nfs <ip>:<nfsDir> </mnt/<dirName>
$ cd </mnt/dirName>
```
## Exploit

After the enumeration comes the exploitation of our findings on the attack vectors.

A zip file was found in the NFS server, which required a password. The best tool that comes in use here is John the ripper

```console
$ zip2john save.zip > zip.hash
$ john --wordlist=rockyou.txt zip.hash
```
This gave the password for the .zip file and it contained two files that were really essential for rooting the system
- id_rsa
- todolist.txt
The id_rsa is a private key that can be used for login into a system through ssh without a password.

But before we try to ssh into the system we needed a username, here our exploit for the Boltwire LFI will come in handy.

```console
GET http://<ip>:8080/dev/
```

- It was a very basic LFI Parameter vulnerability which could be quered through a HTTP GET request whilst being a authenticated user.
- Can be done through either curl or postman or just directly injecting the 

## Conclusion