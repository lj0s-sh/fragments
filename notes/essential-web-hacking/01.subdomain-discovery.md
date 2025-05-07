# 01.subdomain-discovery

02/05/25

## Recall

- What is subdomain discovery?
- Why is it important?
- What are the types of subdomain discovery?
- What is passive enumeration?
- What is active enumeration?
- How to perform subdomain discovery in practice?

## Notes

Subdomain discovery is a reconnaissance technique used to identify subdomains belonging to a target domain. It plays a crucial role in understanding the infrastructure behind a web application, often revealing internal services, forgotten staging environments, and less monitored assets. These subdomains can sometimes expose vulnerable services that would otherwise remain hidden during surface-level scans.

The process can be divided into two main approaches: 

1. **Passive enumeration**
2. **Active enumeration**

### Passive enumeration

Passive subdomain enumeration collects data from public sources without interacting directly with the target. This makes it a stealthier option, as it does not generate logs on the target's infrastructure. Sources such as the Wayback Machine, SSL certificate transparency logs, and platforms like Security Trails or crt.sh are commonly used. Tools like subfinder automate the aggregation of these sources.

Example using Subfinder:

```bash
subfinder -d "nubank.com.br" -silent
```

This command queries multiple public databases and returns a list of known subdomains. However, this method is limited to what has already been indexed or published, so private or newly created subdomains may not appear.

Output:

```bash
prod-s7-stormshield.nubank.com.br
prod-s2-zapdos.nubank.com.br
prod-s4-shore.nubank.com.br
novidades.nubank.com.br
prod-s3-chateau.nubank.com.br
prod-s13-tempranillo.nubank.com.br
prod-s18-moises.nubank.com.br
prod-s5-facade.nubank.com.br
prod-s8-customers.nubank.com.br
prod-s12-blade-runner.nubank.com.br
portal-srv2-br.noc.nubank.com.br
prod-s16-stormshield.nubank.com.br
prod-s14-relampago-marquinhos.nubank.com.br
[...]
```

### Active enumeration

Active subdomain enumeration sends DNS requests directly to the target's DNS servers or resolvers in order to brute-force subdomains based on a predefined wordlist. Unlike passive methods, this approach is more aggressive and may generate traffic or be logged by monitoring systems. Tools like dnsbruter or dnsx are often used for this purpose.

Example using DNSBruter:

```bash
dnsbruter -s -d nubank.com.br -w subdomain-wordlist.txt 
```

The tool tests each word in the list by prepending it to the domain and sending a DNS query. Valid responses indicate that the subdomain exists and may be serving content or exposing services. These results can reveal both user-facing services and internal tools or environments that might be vulnerable or misconfigured.

Output:

```bash
blog.nubank.com.br
www.blog.nubank.com.br
images.nubank.com.br
vpn.nubank.com.br
email.nubank.com.br
app.nubank.com.br
webdisk.blog.nubank.com.br
autoconfig.blog.nubank.com.br
autodiscover.blog.nubank.com.br
cdn.nubank.com.br
content.nubank.com.br
testing.nubank.com.br
jira.nubank.com.br
share.nubank.com.br
link.nubank.com.br
account.nubank.com.br
[...]
```

## Summary

Subdomain discovery is a fundamental technique in reconnaissance, used to map the infrastructure surrounding a target application. It is characterized by the identification of domain prefixes through passive queries or active DNS brute-forcing. Passive enumeration gathers subdomains from public datasets like SSL certificates and archived web pages, while active enumeration probes DNS servers directly to find unlisted assets. By revealing forgotten or unprotected subdomains, this process expands the attack surface and increases the chances of finding exploitable weaknesses during further analysis.