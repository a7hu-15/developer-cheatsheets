# 🐧 Linux Bash & Shell Scripting Cheatsheet

A production reference guide for Linux CLI commands, shell scripting syntax, text manipulation (`awk`, `sed`, `grep`), process management, and environment variables.

---

## ⚡ File System & Navigation

```bash
# Display disk usage and free space in human-readable format
df -h
du -sh /var/log/*

# List files sorted by size or modification time
ls -laS    # Sort by size descending
ls -latr   # Sort by time ascending (newest at bottom)

# Find files modified in the last 24 hours
find /var/log -type f -mtime -1 -name "*.log"

# Search for executables in PATH
which python3
whereis nginx
```

---

## 🔎 Text Processing: `grep`, `awk`, `sed`

```bash
# Grep: Search pattern recursively with line numbers and context
grep -rnI "ERROR" /var/log/app/ --context=2

# Awk: Extract specific columns (e.g. print IP and HTTP status from web logs)
awk '{print $1, $9}' access.log | sort | uniq -c | sort -nr | head -n 10

# Awk: Sum values in column 3
awk '{sum += $3} END {print "Total:", sum}' data.txt

# Sed: Search and replace inline in file
sed -i 's/OLD_HOST/NEW_HOST/g' config.env

# Sed: Remove blank lines or comments
sed -i '/^#/d; /^$/d' server.conf
```

---

## ⚙️ Process Management & Monitoring

```bash
# View active processes sorted by memory or CPU usage
ps aux --sort=-%mem | head -n 10
ps aux --sort=-%cpu | head -n 10

# Search for process ID by name
pgrep -fl "node"

# Send termination signals
kill -15 <PID>    # SIGTERM (graceful shutdown)
kill -9 <PID>     # SIGKILL (forceful termination)

# Run process in background detached from terminal
nohup python3 worker.py > worker.log 2>&1 &
```

---

## 📜 Bash Scripting Boilerplate & Control Flow

```bash
#!/usr/bin/env bash
set -euo pipefail  # Exit on error, unset variable, or pipe failure

# Variables and Environment
READ_ONLY_VAR="PROD"
readonly READ_ONLY_VAR

# Conditional Statement
if [[ -f "/etc/app.conf" && -r "/etc/app.conf" ]]; then
    echo "Configuration file found and readable."
else
    echo "Error: Configuration file missing!" >&2
    exit 1
fi

# Loop over items
for file in *.txt; do
    [[ -e "$file" ]] || continue
    echo "Processing $file..."
done

# Function with positional parameters
log_message() {
    local level="$1"
    local msg="$2"
    echo "[$(date +'%Y-%m-%dT%H:%M:%S')] [$level] $msg"
}

log_message "INFO" "Script execution completed successfully."
```

---

## 🔌 Networking & Port Inspection

```bash
# Inspect open listening ports and associated processes
netstat -tulpn
ss -tulpn

# Test HTTP connection and view response headers
curl -I -L https://api.example.com/health

# Track network route hops to target host
traceroute example.com
mtr example.com
```
