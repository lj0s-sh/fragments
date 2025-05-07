# 02.application-discovery

03/05/25

## **Recall**

- What is application discovery?
- Why is it important?
- What are the phases of application discovery?
- What is URL discovery?
- What is parameter discovery?
- What is content discovery?
- What is fuzzing?
- What is port scanning?

## **Notes**

### Application discovery

**Application Discovery** is the technique of identifying information from a target application, such as its URLs, input parameters, response codes, etc. It is part of the reconnaissance phase and helps uncover vulnerabilities such as XSS or SQL Injection.

Application discovery can be broken down into three main phases:

1. **URL discovery**
2. **Parameter discovery**
3. **Content discovery**

Additional complementary techniques often include:

- **Fuzzing**
- **Port Scanning**

### **URL Discovery**

URL discovery focuses on identifying all endpoints exposed by the target application. Tools like gau or waybackurls collect these URLs from public sources such as the Wayback Machine, Common Crawl, or URL indexing services.

Example using GaU:

```bash
echo '10.10.10.1' | gau
```

The collected URLs may include login pages, administrative routes, or deprecated paths, which can reveal interesting targets for further analysis.

Output:

```bash
<http://10.10.10.1:80/>
<https://10.10.10.1/%22/%3E%3C/head%3E%3C/html%3E>
<http://10.10.10.1/$527,600,00>
<http://10.10.10.1/$527,600,000>
<https://10.10.10.1/$527,600,000180328>
[...]
```

### **Parameter Discovery**

After mapping the URLs, the next step is to enumerate query parameters that may be associated with them. These parameters can serve as entry points for attacks like XSS or SQLi, especially when the application reflects or processes the input insecurely. Tools like ParamSpider and KXSS help automate this enumeration. 

Example using ParamSpider:

```bash
paramspider -d "10.10.10.1"
```

This kind of passive scraping often reveals dozens of parameters passed via GET requests, many of which can be further fuzzed or tested for input-based vulnerabilities.

Output:

```bash
[INFO] Fetching URLs for 10.10.10.1
[INFO] Found 46552 URLs for 10.10.10.1
[...]
<https://www.10.10.10.1/conta/pix?utm_source=FUZZ&utm_medium=FUZZ>
[...]
```

### **Content Discovery**

In content discovery, the goal is to analyze how the application responds to various probes — including which status codes it returns, what server technology it uses, and what page titles or metadata are exposed. You can automate this using tools like httpx.

Example using httpx:

```bash
cat urls.txt | httpx -sc -title -server -ip -probe -silent
```

This phase can help identify resources that return 403 or 404 codes, but still reveal important backend infrastructure details, headers, or patterns in error responses.

Output:

```bash
<https://10.10.10.1/_next/static/>... [SUCCESS] [404] [Error Page | Nubank] [istio-envoy] [IP]
[...]
```

### **Fuzzing**

Fuzzing is often used as a complementary technique to brute-force hidden files, directories, or parameter values. By supplying a wordlist, tools like wfuzz, ffuf, or dirsearch attempt to find valid paths that are not directly linked in the application. 

Example using Wfuzz:

```bash
wfuzz -c -z file,fuzzing-wordlist.txt <https://blog.10.10.10.1/FUZZ>
```

If the server responds with anything other than a 404, it may indicate a valid (but hidden) resource.

Output:

```bash
Target: <https://blog.10.10.10.1/FUZZ>
[...]
000000006: 403 "api/.env"
000000013: 301 "wp-admin"
[...]
```

### **Port Scanning**

At the infrastructure level, port scanning is used to identify open ports and active services running on the server. This can reveal additional web interfaces, admin panels, or exposed services that are not directly tied to the application layer. The most commonly used tool is nmap:

Example using Nmap:

```bash
sudo nmap 10.10.10.1 -sV
```

The scan output shows service versions and can highlight vulnerable services like outdated web servers, SSH daemons, or exposed APIs.

Output:

```bash
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Amazon CloudFront httpd
443/tcp open  ssl/http Amazon CloudFront httpd
[...]
```

## **Summary**

Application discovery is a foundational technique in reconnaissance, focused on identifying and analyzing the components and behaviors of a target web application. It is characterized by the mapping of URLs, enumeration of parameters, and inspection of server responses to uncover possible entry points. Combined with techniques like fuzzing and port scanning, it enables attackers or testers to develop a complete picture of the application’s attack surface, increasing the chances of identifying critical vulnerabilities early in an assessment.