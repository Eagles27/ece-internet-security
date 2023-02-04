# Report DC-4

## Credit

- **Maxime GILLEN** [maxime.gillen@edu.fr](mailto:maxime.gillen@edu.fr)

## Nmap Scan

> The IP address of the target is **10.0.2.4**

### Command

```bash
nmap -sS -A 10.0.2.4 -oN nmapReport.txt
```

### Nmap Report

```bash
Nmap scan report for 10.0.2.4
Host is up (0.0018s latency).
Not shown: 998 closed tcp ports (reset)
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
| 2048 8d6057066c27e02f762ce642c001ba25 (RSA)
| 256 e7838cd7bb84f32ee8a25f796f8e1930 (ECDSA)
|\_ 256 fd39478a5e58339973739e227f904f4b (ED25519)
80/tcp open http nginx 1.15.10
|\_http-server-header: nginx/1.15.10
|\_http-title: System Tools
MAC Address: 08:00:27:E3:7E:DC (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT ADDRESS
1 1.84 ms 10.0.2.4

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

```

## Work on port 80

We will use **Burp** to BruteForce the login page.

### BruteForce

Firstly we suppose than the login is **admin** and we will use the wordlist given in the subject to find the password.

For the password **happy** we have a different response length than the others, so we can suppose that it's the good one.

> So we can try to login with the credentials **admin:happy** and we have a successfull login.

## Find a way to get a shell

When we are logged in, we can see that we are on the page **/command.php**. By using for example the repeater of Burp, we can see that the page is vulnerable to **Command Injection**.

### Command Injection

The goal is to obtain the list of users on the server. To do it we have 2 methods:

- **Method 1**: We can use the command **cat /etc/passwd** to get the list of users.
- **Method 2**: We can find a way to get a reverse shell.

### Method 1

- We can use the command **ls /home** to get the list of users.

### Method 2

- We have to find a php vulnerability to get a reverse shell. To do it we just have to understand how the command is executed. To do it we just have to cat the file **command.php**.

- And we can see than commands use the function : **php shell exec**

- We can generate a reverse shell [on this website](https://www.revshells.com/) for example.

- Now we just have to add a listener on our machine with the command : `nc -nlvp [The Port than we want listen]`

- And we can send the reverse shell with Burp.

- And we have a shell.

- `cat /etc/passwd` to get the list of users. And we can see that we have 3 users **charles**, **jim** and **sam**.
