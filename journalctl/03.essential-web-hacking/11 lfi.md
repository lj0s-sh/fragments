# 11.lfi

14/05/25

## Recall

- What is LFI (Local File Inclusion)?
- How does LFI typically occur in web applications?
- What can an attacker do with LFI?
- What techniques can bypass simple filtering in LFI?

## Notes

### What it is

**Local File Inclusion (LFI)** is a web vulnerability that occurs when a web application includes files from the server’s filesystem without properly validating user input. This often happens when the application dynamically builds file paths using untrusted parameters.

If the input is not sanitized, attackers can manipulate the file path to read sensitive files, and in some cases, execute code or escalate privileges. 

### How to identify it

To identify a LFI vulnerability, look for parameters in URLs that suggest dynamic file loading:

```bash
https://example.com/index.php?page=home
```

File fields, such as images or documents, might accept LFI aswell

```html
<img src="images/profile-photo.png" />
```

Once identified, begin injecting LFI payloads. These payloads consist of file paths. If the application **doesn't restrict inclusion to a specific directory** (e.g., no hardcoded prefix like `images/`), you can freely specify the path:

```html
https://example.com/index.php?page=/etc/passwd
```

However, if the application **does prepend a folder path** (e.g., `images/`), you’ll need to use **path traversal** (`../`) to escape the restricted directory and reach the desired file:

```bash
<img src="images/../../../../etc/passwd" />
```

If the application returns file content ( `/etc/passwd` on Unix), then the vulnerability it's likely present.

### How to exploit it

The primary goal of a Local File Inclusion vulnerability is to read sensitive files on the server:

```html
https://target.com/index.php?page=../../../../etc/passwd
```

This can expose critical information such as system users or application configs. Common targets include: `/etc/passwd` (user list), `~/.bash_history` (command history), `~/.ssh/id_rsa` or `id_dsa` (private SSH keys), `config.php`, `.env` (application secrets)

## Summary

Local File Inclusion (LFI) occurs when user input is used to construct file paths that the server includes without validation. This can allow attackers to read local files, extract sensitive data, or even execute code under certain conditions (log poisoning). The impact of LFI varies from simple information disclosure to full remote code execution.