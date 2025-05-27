# 09.crlf-injection

14/05/25

## Recall

- What is CRLF (Carriage Return Line Feed)?
- How is CRLF used in HTTP communication?
- What happens when CRLF characters are injected into a request?
- What can an attacker achieve through CRLF Injection?

## Notes

### What it is

**CRLF (Carriage Return Line Feed)** refers to the two-character sequence `\r\n`, which represents a new line in many network protocols, including HTTP. When you press “Enter” on a keyboard, the carriage return (`\r`) returns the cursor to the beginning of the line, and the line feed (`\n`) moves it to the next line.

In the context of HTTP, CRLF characters are used to separate the **request line**, **headers**, and **body**. Here's a simplified structure of an HTTP request:

```bash
METHOD URI HTTP/VERSION
Header-1: value
Header-2: value

Request body (for POST, PUT, etc.)
```

If a web application includes user input in headers without sanitizing CRLF sequences, an attacker may inject malicious headers or split the HTTP response. This is known as **CRLF Injection**, and it can lead to **header injection**, **cookie poisoning**, **open redirection**, or even **HTTP response splitting** which can lead to other injections attacks such as **XSS** and **HTML Injection**.

### How to identify it

To identify a CRLF injection vulnerability, observe how the server responds when you inject CRLF payloads in query parameters or other inputs. You can craft such payloads by URL-encoding `\r` as `%0d` and `\n` as `%0a`.

Example:

```bash
GET /%0d%0a%0d%0a HTTP/1.0
Host: 10.10.0.4
```

If the server does not sanitize the CRLF characters, the response may be splitted:

```bash
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.18.0
Date: Tue, 13 May 2025 13:06:32 GMT
Content-Type: text/html
Content-Length: 145
Connection: close
Location: http://10.10.0.4/

admin/
```

Another sign is if the response structure breaks, for example, by inserting a second `Location` header or injecting headers, which would confirm that CRLF injection is possible.

### How to exploit it

Once the vulnerability is confirmed, the attacker can exploit it to inject malicious headers (`Set-Cookie`, `Location`, `X-Forwarded-For`), poison response behavior (redirect victims), bypass access controls and much more.

After confirming the vulnerability, the attacker looks for sensitive actions that can be triggered via authenticated requests — such as **changing a user’s password** or **modifying profile settings**.

Example with cookie injection:

```bash
GET /%0d%0aSet-Cookie:%20auth=admin HTTP/1.0
Host: 10.10.0.4
```

Response:

```bash
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.18.0
Date: Tue, 13 May 2025 13:06:32 GMT
Content-Type: text/html
Content-Length: 145
Connection: close
Location: http://10.10.0.4/
Set-Cookie: auth=admin
```

Now the victim’s browser stores a malicious cookie, which may be used for **session fixation**, **session hijacking** etc.

## Summary

CRLF Injection is a vulnerability that occurs when user input is improperly included in HTTP responses without filtering newline characters (`\r\n`). By injecting CRLF sequences, attackers can manipulate HTTP headers, poison cookies, trigger redirects, or even split responses. This attack can lead to session manipulation, cache poisoning, and open redirection — especially when combined with other vulnerabilities. Prevention requires strict sanitization of user input, particularly when reflected in HTTP headers, and rejecting or encoding any untrusted CRLF characters.