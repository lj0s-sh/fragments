# 13.unrestricted-file-upload

24/05/25

## Recall

- What is an Unrestricted File Upload vulnerability?
- Why does allowing unrestricted uploads often lead directly to RCE?
- What types of validation failures commonly lead to file upload vulnerabilities?
- What are common techniques attackers use to bypass file upload restrictions?
- What configurations can prevent execution of uploaded files?
- How can developers securely implement file uploads?

## Notes

### What it is

The **Unrestricted File Upload** vulnerability happens when a web application allows users to upload files without proper validation. This flaw might seem simple at first, but in practice, it’s one of the most direct and dangerous paths to achieving **Remote Code Execution (RCE)**. If the server accepts a file that it can execute—such as PHP, ASP, JSP, or others—the attacker can run arbitrary commands remotely.

This vulnerability becomes critical when the uploaded file is stored in a web-accessible directory and the server treats it as executable. In that case, the attacker can upload a malicious script that functions as a WebShell, allowing them to execute commands through HTTP requests.

### How to identify it

Identifying an Unrestricted File Upload vulnerability starts by analyzing how the upload functionality handles user input. If the application only checks the file extension superficially —l ike allowing files that end with `.jpg`, `.png`, or `.pdf` without inspecting the actual content — this is already a sign of weakness. It’s common for developers to rely on client-side validation (like HTML or JavaScript) or simple server-side checks that can be easily bypassed by attackers.

A typical test involves uploading a basic payload like this:

```php
<?php system($_GET["cmd"]); ?>
```

If the application accepts the file and stores it in a directory such as `/uploads/`, `/images/`, or `/files/` without verifying whether the file contains executable code, the vulnerability is likely present.

The next step is to try accessing it directly via URL:

```bash
https://target.com/uploads/shell.php?cmd=id
```

If this returns the result of the `id` command, then the server is executing the file, confirming that RCE is possible.

Even if the application seems to restrict certain file types, there are well-established techniques to bypass those filters, such as **double extension**, **MIME type tampering** and **magic bytes confusion**.

### How to exploit it

Exploiting an Unrestricted File Upload vulnerability is pretty straightforward. Once confirming that a file upload is possible and that the server executes it, the attacker can upload a simple **WebShell** like this:

```bash
<?php system($_GET["cmd"]); ?>
```

## Summary

**Unrestricted File Upload** is a high-impact vulnerability that occurs when a web application accepts file uploads without adequate validation. This flaw often leads directly to **Remote Code Execution (RCE)** if the uploaded file is processed in an executable context.

Attackers can exploit this by uploading WebShells, deploying reverse shells, and gaining complete control over the server within the permissions of the web service user. Effective exploitation allows for privilege escalation, persistence, lateral movement, and data exfiltration.

Proper mitigation requires a combination of validating file types, restricting file storage locations, disabling execution in upload directories, and enforcing robust security checks server-side.