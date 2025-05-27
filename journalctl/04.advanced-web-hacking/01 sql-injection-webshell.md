# 01.sql-injection-webshell

26/05/25

## Recall

- What is an SQL Injection WebShell?
- How does SQL Injection lead to remote code execution via file writing?
- What conditions must be present on the target for a WebShell via SQLi to succeed?
- How can an attacker confirm if the server allows file writes?
- How can developers prevent SQL Injection WebShell attacks?

## Notes

### What it is

An **SQL Injection WebShell** is an attack technique where an attacker leverages an SQL Injection vulnerability to write a malicious file (usually a webshell) to the target server's web root. This is made possible using SQL file output functions such as `INTO OUTFILE` in MySQL.

If the database server has the necessary file system permissions, the attacker can write a file containing server-side code (like PHP). Once placed in the web-accessible directory, this file acts as a webshell, granting the attacker remote code execution (RCE) capabilities through the web interface.

This technique effectively escalates an SQL Injection vulnerability into full remote code execution, significantly increasing the impact.

### How to identify it

The identification process involves two stages:

1. **Find an SQL Injection:**
    
    This is done by injecting classic payloads (`'`, `' OR 1=1 --`, `' UNION SELECT ...`) to check if the application is vulnerable to SQL Injection.
    
2. **Test for file write permissions:**
    
    Once SQL Injection is confirmed, test whether the database server has permissions to write files to the web root or other accessible directories.
    

Example payload for testing file write via SQL Injection:

```bash
' UNION SELECT 1, 2, ..., "<?php system(\"id\") ?>" INTO OUTFILE "/var/www/html/payload.php"; -- -
```

After sending this payload, check if the file was created by visiting:

```bash
https://www.target.com/payload.php
```

If the payload executes (e.g., shows the output of `id`), the server is vulnerable to an SQL Injection WebShell.

### How to exploit it

Once confirmed, the exploitation consists of writing a webshell that allows executing arbitrary commands.

Example PHP webshell payload:

```bash
' UNION SELECT 1, 2, ..., "<?php system($_GET[\"cmd\"]) ?>" INTO OUTFILE "/var/www/html/webshell.php"; -- -
```

The attacker can now interact with the webshell:

```bash
https://www.target.com/webshell.php?cmd=whoami
```

This allows command execution on the server. From here, the attacker can escalate further by spawning a reverse shell, adding users, pivoting inside the network, or exfiltrating data.

## Summary

**SQL Injection WebShell** is an attack that escalates a typical SQL Injection vulnerability into remote code execution by writing a malicious file (webshell) to the web server through the database. This is achieved using functions like `INTO OUTFILE` in MySQL, assuming the server has permissive file system access.

Once deployed, the webshell allows attackers to execute system commands remotely, significantly amplifying the severity of the initial vulnerability. Exploiting this requires both an SQL Injection point and file write capabilities on the database server.