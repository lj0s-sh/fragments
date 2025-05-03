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

**Application Discovery** is the technique of identifying information from a target application, such as its URLs, input parameters, response codes, etc. It is part of the reconnaissance phase and helps uncover vulnerabilities such as XSS or SQL Injection.

Application discovery can be broken down into three main phases:

1. **URL discovery**
2. **Parameter discovery**
3. **Content discovery**

### **URL Discovery**

- **Definition**: The process of discovering URLs associated with the application.
- **Goal**: Understand the app's structure and expose internal service URLs.
- **Tools**: GaU
    - Example using GaU:
    
    ```bash
    echo 'www.nubank.com.br' | gau
    ```
    
    - **Result**:
        - URLs are collected from passive queries to public sources and may include login pages, internal routes, etc.
        - **Output**:
        
        ```bash
        <http://www.nubank.com.br:80/>
        <https://nubank.com.br/%22/%3E%3C/head%3E%3C/html%3E>
        <http://www.nubank.com.br/$527,600,00>
        <http://www.nubank.com.br/$527,600,000>
        <https://www.nubank.com.br/$527,600,000180328>
        [...]
        ```
        

### **Parameter Discovery**

- **Definition**: Identifying parameters associated with application URLs.
- **Goal**: Enumerate parameters to detect vulnerable ones like XSS or SQLi.
- **Tools**: ParamSpider, KXSS
    - Example using ParamSpider:
    
    ```bash
    paramspider -d "nubank.com.br"
    ```
    
    - **Result**:
        - Parameters are extracted from public sources and might include reflected or unfiltered values.
        - **Output**:
        
        ```bash
        [INFO] Fetching URLs for nubank.com.br
        [INFO] Found 46552 URLs for nubank.com.br
        [...]
        <https://www.nubank.com.br/nu/conta/pix?utm_source=FUZZ&utm_medium=FUZZ>
        [...]
        ```
        

### **Content Discovery**

- **Definition**: Understanding the behavior of an application by analyzing response status codes, server info, etc.
- **Goal**: Identify behaviors and vulnerabilities in the app environment or file structure.
- **Tools**: HTTPX, Aquatone
    - Example using HTTPX:
    
    ```bash
    cat urls.txt | httpx -sc -title -server -ip -probe -silent
    ```
    
    - **Result**:
        - Each URL is probed and response data is collected.
        - **Output**:
        
        ```bash
        <https://nubank.com.br/_next/static/>... [SUCCESS] [404] [Error Page | Nubank] [istio-envoy] [IP]
        [...]
        ```
        

In addition to these phases, application discovery often includes two other techniques:

1. **Fuzzing**
2. **Port Scanning**

### **Fuzzing**

- **Definition**: Automated process for testing directory, file, or parameter names.
- **Goal**: Discover unmapped resources via brute force.
- **Tools**: Wfuzz, Fuff
    - Example using Wfuzz:
    
    ```bash
    wfuzz -c -z file,fuzzing-wordlist.txt <https://blog.nubank.com.br/FUZZ>
    ```
    
    - **Result**:
        - Each wordlist entry is tested to see if it resolves to a real resource.
        - **Output**:
        
        ```bash
        Target: <https://blog.nubank.com.br/FUZZ>
        [...]
        000000006: 403 "api/.env"
        000000013: 301 "wp-admin"
        [...]
        ```
        

### **Port Scanning**

- **Definition**: Identifying open or closed ports on a server.
- **Goal**: Reveal active services and potential targets.
- **Tools**: Nmap
    - Example using Nmap:
    
    ```bash
    sudo nmap nubank.com.br -sV
    ```
    
    - **Result**:
        - Lists open ports and service versions (depending on flags used).
        - **Output**:
        
        ```bash
        PORT    STATE SERVICE  VERSION
        80/tcp  open  http     Amazon CloudFront httpd
        443/tcp open  ssl/http Amazon CloudFront httpd
        [...]
        ```
        

## **Summary**

Application discovery is the process of identifying information about a web application to aid in the reconnaissance phase. It includes three core phases:

- **URL Discovery** (identify endpoints)
- **Parameter Discovery** (identify input points)
- **Content Discovery** (analyze server behavior)

It also leverages:

- **Fuzzing** to brute-force hidden paths, files, or parameters
- **Port Scanning** to map exposed services on the host