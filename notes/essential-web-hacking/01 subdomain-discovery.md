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

**Subdomain discovery** is an enumeration technique used to identify subdomains of an application. It is performed during the reconnaissance phase and helps uncover the application's infrastructure, including internal or hidden subdomains.

There are two main approaches to **subdomain discovery**:

1. **Passive enumeration**
2. **Active enumeration**

### Passive enumeration

- **Definition**: Enumeration of subdomains without directly interacting with the target. This method relies on public information sources. Issued SSL certificates can also be consulted.
- **Goal**: To avoid generating logs or triggering detection mechanisms on the target.
- **Tools**: Public sources such as [Security Trails](https://securitytrails.com/), [Web Archive](https://web.archive.org/), certificate repositories like [crt.sh](https://crt.sh/), and automated tools like Subfinder.
    - Example of passive enumeration using Subfinder:
    
    ```bash
    subfinder -d "nubank.com.br" -silent
    ```
    
    - **Result**:
        - Subfinder searches across multiple sources, including Security Trails and certificate databases, to return known subdomains. However, unmapped or private subdomains might not appear through this method.
        - **Output**:
        
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

- **Definition**: Enumeration of subdomains through direct interaction with the target. This method sends multiple requests to the target's DNS servers.
- **Goal**: To identify subdomains that are not publicly listed.
- **Tools**: DNSBruter
    - Example of active enumeration using DNSBruter
    
    ```bash
    dnsbruter -s -d nubank.com.br -w subdomain-wordlist.txt 
    ```
    
    - **Result**:
        - The tool validates each subdomain from the wordlist by sending a high volume of DNS queries to the target.
        - **Output**
        
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

Subdomain discovery is a technique for identifying subdomains of a target application. It can be performed using historical data, issued digital certificates, or brute-forcing techniques. By mapping out subdomains, it's possible to better understand the application's infrastructure and potentially find vulnerable or exposed components.