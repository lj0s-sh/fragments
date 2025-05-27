# 15.idor

24/05/25

## Recall

- What is an IDOR vulnerability?
- Why does IDOR happen, even when authentication is properly implemented?
- How can an attacker identify and exploit an IDOR?
- What types of objects are commonly exposed via IDOR?
- How does IDOR impact APIs and RESTful services?
- How can developers effectively prevent IDOR vulnerabilities?

## Notes

### What it is

**IDOR (Insecure Direct Object Reference)** is a vulnerability that occurs when an application allows users to access, modify, or delete data simply by manipulating identifiers in requests, without verifying whether the user actually has permission to access that resource. In essence, the server trusts that if a user knows the ID of an object — whether it's a file, user profile, invoice, or order — they should be allowed to interact with it. This trust is the vulnerability itself.

### How to identify it

Finding IDOR vulnerabilities usually involves analyzing how the application references objects—whether through URL parameters, JSON bodies, or query strings. When an API endpoint includes an object identifier, it’s worth checking whether changing that identifier grants access to objects belonging to other users.

For example, if a user’s profile is accessible via:

```bash
https://target.com/user/1001
```

Simply changing the ID to `1002`:

```bash
https://target.com/user/1002
```

…might reveal someone else’s profile if there is no authorization check on the backend. 

Similarly, file download links like:

```bash
/invoices/2024-001.pdf
```

…can become:

```bash
/invoices/2024-002.pdf
```

and, if vulnerable, allow the download of someone else’s document.

This applies not only to URL parameters but also to POST and PUT requests where object IDs might be included in the request body:

```bash
{
  "user_id": 1002,
  "action": "update_profile"
}
```

When APIs rely solely on **authentication** but lack proper **authorization** (checking whether the user is allowed to interact with a specific resource), IDOR is almost inevitable.

### How to exploit it

Exploiting an IDOR is as straightforward as interacting with the application as intended, but then modifying the object identifiers in requests to see if unauthorized data or functionality becomes accessible. There is no need for special payloads, injections, or encoding tricks. The attacker is simply leveraging the application's own API or URL structure.

For example, after authenticating as a normal user, an attacker accesses:

```bash
GET /api/orders/123/
```

…and then changes it to:

```bash
GET /api/orders/124/
```

If order 124 belongs to a different user but is still returned, the application is vulnerable to IDOR. The same applies to edit or delete operations, which can have an even greater impact. 

IDOR can also affect more than just data retrieval. Attackers can often modify or delete other users’ data or perform critical actions intended to be restricted.

## Summary

**Insecure Direct Object Reference (IDOR)** is a vulnerability that occurs when an application allows users to directly reference objects, such as files, database entries, or records, using identifiers exposed in the request. When the backend fails to verify whether the authenticated user is authorized to access that object, attackers can manipulate those identifiers to gain unauthorized access to data or actions.

Despite being conceptually simple, IDOR can lead to severe consequences, including privacy violations, data manipulation, and even full account takeover in certain conditions. This vulnerability is particularly prevalent in API-driven applications, where developers may focus on authentication while neglecting proper object-level authorization.

Preventing IDOR requires strict backend validation to ensure that every object interaction is checked against the permissions of the authenticated user. This is done by implementing object-level permissions, either manually or via frameworks, and designing APIs with the principle of least privilege in mind.