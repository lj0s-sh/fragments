# 16.subdomain-takeover

26/05/25

## Recall

- What is a Subdomain Takeover vulnerability?
- Why does Subdomain Takeover happen?
- How can an attacker identify and confirm a vulnerable subdomain?
- How does DNS configuration play a role in this vulnerability?
- What are common cloud providers involved in subdomain takeover scenarios?
- What steps can developers take to prevent Subdomain Takeover?

## Notes

### What it is

**Subdomain Takeover** is a vulnerability that occurs when a subdomain points to an external service (like AWS S3, GitHub Pages, Heroku, or Azure) that is no longer claimed or configured by the organization. While the DNS entry (typically a CNAME) remains active, the resource it points to has been deleted or is unassigned.

This creates an opportunity for an attacker to register the missing resource—such as creating a bucket, repository, or cloud service with the same name—and effectively take control of the subdomain. Once claimed, the attacker can serve any content through that domain, including phishing pages, malware, or malicious scripts.

Subdomain Takeover is particularly common in scenarios where companies set up temporary resources (for campaigns, prototypes, or microsites) and forget to remove the associated DNS entries after the resource is decommissioned.

### How to identify it

The first step is to enumerate the target's subdomains.

Subdomain discovery using Subfinder:

```bash
subfinder -d "target.com"
```

Output:

```bash
www.target.com
mail.target.com
dev.target.com
```

Once you have a list of subdomains, check whether any of them have a CNAME record pointing to an external service.

Checking for CNAME entries using dig:

```bash
dig CNAME dev.target.com +short
```

Output:

```bash
static-files.s3-website-us-east-1.amazonaws.com
```

This indicates that `dev.target.com` is pointing to an AWS S3 bucket configured as a website.

The next step is to verify whether the destination service still exists. Visit the URL:

```bash
http://static-files.s3-website-us-east-1.amazonaws.com
```

If the response contains an error such as:

```bash
<Code>NoSuchBucket</Code>
<Message>The specified bucket does not exist</Message>
```

or if the page shows a generic "404 Not Found" related to the cloud provider, the subdomain is very likely vulnerable to takeover.

### How to exploit it

Once a subdomain is found pointing to an unclaimed resource, the exploitation process involves three steps: registering the missing resource by creating the corresponding bucket, repository, or cloud instance with the same name used in the DNS record; verifying that the subdomain properly resolves to the newly created resource; and finally, serving malicious content, such as phishing pages, malware, or any other payload through the compromised subdomain.

## Summary

**Subdomain Takeover** is a vulnerability that occurs when a DNS entry (commonly a CNAME) continues to point to an external service that has been removed, deleted, or left unclaimed. An attacker can register the missing resource on that service and take control of the subdomain, serving malicious content under the trusted domain.

This vulnerability is particularly common with cloud services like AWS S3, GitHub Pages, Heroku, Azure, Shopify, and similar SaaS platforms. The impact ranges from phishing attacks and malware distribution to severe brand damage and data leakage.

To prevent subdomain takeover, organizations must implement strict DNS hygiene: removing unused DNS records, monitoring dangling DNS entries, and validating that all CNAMEs or service links point to active and controlled resources.