# Essential Linux Commands & Shell Scripting

## ১. কেন দরকার? (Why?)

Kubernetes troubleshoot করতে হলে Linux command line এ comfortable হতে হবে।

```bash
# Pod এ ঢুকলে:
kubectl exec -it mypod -- bash

# এখন কী করবে?
# - File খুঁজতে হবে
# - Log parse করতে হবে
# - Config file edit করতে হবে
# - Network test করতে হবে
```

**Command line = DevOps এর primary interface!**

---

## ২. Text Processing Commands

### grep - Search in Files

**Pattern matching এবং searching এর জন্য সবচেয়ে important tool।**

```bash
# Basic search
grep "error" app.log

# Case insensitive
grep -i "error" app.log

# Line numbers
grep -n "error" app.log

# Output:
45:ERROR: Database connection failed
78:ERROR: Timeout occurred

# Invert match (lines without pattern)
grep -v "debug" app.log

# Show 2 lines before and after
grep -C 2 "error" app.log

# Recursive search in directory
grep -r "TODO" /path/to/code/

# Count matches
grep -c "error" app.log
# Output: 15

# Show only filenames
grep -l "error" *.log

# Multiple patterns
grep -E "error|warning" app.log
grep "error\|warning" app.log

# Regex
grep -E "^[0-9]+" app.log       # Lines starting with numbers
grep -E "[0-9]{3}-[0-9]{3}" app.log  # Phone pattern
```

### awk - Text Processing Language

**Column-based data processing এর জন্য powerful।**

```bash
# Print specific column
echo "apple banana cherry" | awk '{print $2}'
# Output: banana

# CSV processing
cat users.csv
# name,age,city
# John,25,NYC
# Alice,30,LA

awk -F',' '{print $1, $3}' users.csv
# Output:
# name city
# John NYC
# Alice LA

# With conditions
awk -F',' '$2 > 25 {print $1}' users.csv
# Output: Alice

# Sum column
ps aux | awk '{sum += $4} END {print sum}'
# Sums %MEM column

# Format output
awk '{printf "%-10s %5d\n", $1, $2}' file.txt
```

**Real Examples:**

```bash
# Get running processes memory
ps aux | awk '{print $11, $4}' | sort -k2 -rn | head -10

# Parse nginx access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
# IPs with request count

# CPU usage per process
ps aux | awk '{if ($3 > 10) print $11, $3}'
# Processes using >10% CPU
```

### sed - Stream Editor

**Text find-and-replace এবং manipulation।**

```bash
# Replace first occurrence
echo "hello world" | sed 's/world/universe/'
# Output: hello universe

# Replace all occurrences (g = global)
echo "cat cat cat" | sed 's/cat/dog/g'
# Output: dog dog dog

# Replace in file (in-place)
sed -i 's/old/new/g' config.txt

# Delete lines
sed '/debug/d' app.log          # Delete lines with 'debug'
sed '1d' file.txt               # Delete first line
sed '1,5d' file.txt             # Delete lines 1-5

# Print specific lines
sed -n '10,20p' file.txt        # Print lines 10-20
sed -n '/error/p' app.log       # Print lines with 'error'

# Multiple operations
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt

# Backup before replacing
sed -i.bak 's/old/new/g' file.txt
# Creates file.txt.bak
```

**Real Examples:**

```bash
# Change config
sed -i 's/DEBUG=true/DEBUG=false/' .env

# Add line after pattern
sed '/\[database\]/a host=localhost' config.ini

# Comment out lines
sed -i '/dangerous_option/s/^/#/' config.txt

# Extract between patterns
sed -n '/START/,/END/p' file.txt
```

---

## ৩. File Operations

### find - Search for Files

```bash
# Find by name
find /path -name "*.log"

# Case insensitive
find /path -iname "*.LOG"

# Find by type
find /path -type f          # Files
find /path -type d          # Directories
find /path -type l          # Symbolic links

# Find by size
find /path -size +100M      # Larger than 100MB
find /path -size -10k       # Smaller than 10KB

# Find by modification time
find /path -mtime -7        # Modified in last 7 days
find /path -mtime +30       # Modified >30 days ago
find /path -mmin -60        # Modified in last 60 minutes

# Find and execute
find /tmp -name "*.tmp" -delete
find /path -name "*.log" -exec gzip {} \;
find /path -type f -exec chmod 644 {} \;

# Find empty files
find /path -empty

# Find by permissions
find /path -perm 777
```

**Real Examples:**

```bash
# Find large log files
find /var/log -name "*.log" -size +100M

# Find config files
find /etc -name "*.conf"

# Find recently modified
find /app -type f -mmin -30

# Clean old tmp files
find /tmp -mtime +7 -delete

# Find and compress logs
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;
```

### wc - Word Count

```bash
# Lines, words, characters
echo "hello world" | wc
# Output: 1  2  12
#         ↑  ↑   ↑
#      lines words chars

# Count lines
wc -l file.txt
cat file.txt | wc -l

# Count words
wc -w file.txt

# Count characters
wc -c file.txt
```

**Real Examples:**

```bash
# Count log entries
wc -l /var/log/syslog

# Count errors
grep "error" app.log | wc -l

# Count running processes
ps aux | wc -l

# Lines of code
find . -name "*.py" | xargs wc -l
```

### cut - Extract Columns

```bash
# By delimiter
echo "apple,banana,cherry" | cut -d',' -f2
# Output: banana

# Multiple fields
echo "a:b:c:d" | cut -d':' -f1,3
# Output: a:c

# Range
echo "a:b:c:d" | cut -d':' -f2-4
# Output: b:c:d

# By character position
echo "hello" | cut -c1-3
# Output: hel
```

**Real Examples:**

```bash
# Extract usernames
cut -d':' -f1 /etc/passwd

# Get IP addresses
ip addr | grep "inet " | cut -d' ' -f6

# Parse CSV
cut -d',' -f1,3 users.csv

# Get process IDs
ps aux | grep nginx | cut -c10-15
```

---

## ৪. Network Commands

### curl - HTTP Client

```bash
# GET request
curl http://example.com

# Save to file
curl -o output.html http://example.com
curl -O http://example.com/file.txt  # Use remote filename

# Follow redirects
curl -L http://example.com

# Verbose output
curl -v http://example.com

# POST request
curl -X POST http://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# PUT request
curl -X PUT http://api.example.com/users/1 \
  -d '{"name":"John Updated"}'

# DELETE request
curl -X DELETE http://api.example.com/users/1

# Authentication
curl -u username:password http://example.com
curl -H "Authorization: Bearer TOKEN" http://api.example.com

# Timeout
curl --connect-timeout 5 --max-time 10 http://example.com

# Show only status code
curl -o /dev/null -s -w "%{http_code}\n" http://example.com
```

**Kubernetes Examples:**

```bash
# Test service
kubectl run curl-test --image=curlimages/curl -it --rm -- sh
> curl http://myservice:80

# Health check
curl http://localhost:8080/health

# API test
curl http://myapp-service/api/users
```

### wget - File Downloader

```bash
# Download file
wget http://example.com/file.zip

# Resume download
wget -c http://example.com/large-file.iso

# Background download
wget -b http://example.com/file.zip

# Recursive download (website)
wget -r -np -k http://example.com/docs/

# Limit speed
wget --limit-rate=200k http://example.com/file.zip

# Try N times
wget --tries=10 http://example.com/file.zip

# Output filename
wget -O myfile.zip http://example.com/download.zip
```

### netstat / ss - Network Statistics

```bash
# Listening ports
netstat -tulpn
ss -tulpn

# Output:
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1234/nginx
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      567/sshd

# Established connections
netstat -an | grep ESTABLISHED
ss -t state established

# Specific port
netstat -tulpn | grep :80
ss -tulpn | grep :80

# Statistics
netstat -s
ss -s
```

---

## ৫. Process Management

### ps - Process Status

```bash
# All processes
ps aux

# Specific user
ps -u username

# Process tree
ps auxf
ps -ejH

# Custom columns
ps -eo pid,ppid,cmd,%cpu,%mem

# Sort by CPU
ps aux --sort=-%cpu | head -10

# Sort by memory
ps aux --sort=-%mem | head -10

# Find process
ps aux | grep nginx

# Or use pgrep
pgrep nginx
pgrep -a nginx  # With command
```

### kill - Send Signals

```bash
# Graceful termination (SIGTERM)
kill 1234

# Force kill (SIGKILL)
kill -9 1234

# Kill by name
pkill nginx
killall nginx

# Kill all processes of user
pkill -u username

# Reload config (SIGHUP)
kill -HUP 1234
pkill -HUP nginx
```

### top / htop - Real-time Monitoring

```bash
# top
top

# Keys:
# P - Sort by CPU
# M - Sort by memory
# k - Kill process
# r - Renice
# q - Quit

# Show specific user
top -u username

# Batch mode (for scripts)
top -b -n 1 > top-output.txt

# htop (better interface)
htop
# F5 - Tree view
# F6 - Sort by
# F9 - Kill
```

---

## ৬. Shell Scripting Basics

### Shebang এবং Basics:

```bash
#!/bin/bash

# This is a comment

# Variables
NAME="John"
AGE=25
echo "Name: $NAME, Age: $AGE"

# Command output in variable
CURRENT_DATE=$(date)
echo "Today is $CURRENT_DATE"

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"
```

### Conditionals:

```bash
#!/bin/bash

# If-else
AGE=20

if [ $AGE -ge 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# If-elif-else
SCORE=75

if [ $SCORE -ge 90 ]; then
    echo "A"
elif [ $SCORE -ge 80 ]; then
    echo "B"
elif [ $SCORE -ge 70 ]; then
    echo "C"
else
    echo "F"
fi

# File checks
FILE="/path/to/file"

if [ -f "$FILE" ]; then
    echo "File exists"
fi

if [ -d "/path/to/dir" ]; then
    echo "Directory exists"
fi

# String comparison
NAME="John"

if [ "$NAME" = "John" ]; then
    echo "Name is John"
fi

if [ -z "$NAME" ]; then
    echo "Name is empty"
fi

# Logical operators
if [ $AGE -ge 18 ] && [ $AGE -le 65 ]; then
    echo "Working age"
fi

if [ $AGE -lt 18 ] || [ $AGE -gt 65 ]; then
    echo "Not working age"
fi
```

### Loops:

```bash
#!/bin/bash

# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# For loop with range
for i in {1..10}; do
    echo "Count: $i"
done

# For loop with array
FRUITS=("apple" "banana" "cherry")
for FRUIT in "${FRUITS[@]}"; do
    echo "Fruit: $FRUIT"
done

# For loop with files
for FILE in *.txt; do
    echo "Processing $FILE"
done

# While loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Read file line by line
while IFS= read -r LINE; do
    echo "Line: $LINE"
done < file.txt

# Infinite loop
while true; do
    echo "Running..."
    sleep 1
done
```

### Functions:

```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"
}

# Call function
greet "John"

# Function with return value
add() {
    local RESULT=$(($1 + $2))
    echo $RESULT
}

SUM=$(add 5 3)
echo "Sum: $SUM"

# Multiple parameters
create_user() {
    local USERNAME=$1
    local EMAIL=$2
    echo "Creating user: $USERNAME ($EMAIL)"
}

create_user "john" "john@example.com"
```

---

## ৭. Practical Shell Scripts

### Script 1: System Info

```bash
#!/bin/bash

# system_info.sh

echo "=== System Information ==="
echo ""

echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2 | tr -d '"')"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo ""

echo "=== CPU Info ==="
lscpu | grep "Model name"
echo ""

echo "=== Memory ==="
free -h
echo ""

echo "=== Disk Usage ==="
df -h /
echo ""

echo "=== Top 5 CPU Processes ==="
ps aux --sort=-%cpu | head -6
echo ""

echo "=== Top 5 Memory Processes ==="
ps aux --sort=-%mem | head -6
```

```bash
chmod +x system_info.sh
./system_info.sh
```

### Script 2: Log Analyzer

```bash
#!/bin/bash

# log_analyzer.sh

LOG_FILE="/var/log/syslog"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file not found!"
    exit 1
fi

echo "=== Log Analysis for $LOG_FILE ==="
echo ""

echo "Total lines: $(wc -l < $LOG_FILE)"
echo ""

echo "Error count: $(grep -ic "error" $LOG_FILE)"
echo "Warning count: $(grep -ic "warning" $LOG_FILE)"
echo ""

echo "=== Last 10 Errors ==="
grep -i "error" $LOG_FILE | tail -10
echo ""

echo "=== Top 10 Most Common Messages ==="
awk '{print $5}' $LOG_FILE | sort | uniq -c | sort -rn | head -10
```

### Script 3: Backup Script

```bash
#!/bin/bash

# backup.sh

SOURCE_DIR="/home/user/data"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

echo "Starting backup..."

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
    echo "Size: $(du -h $BACKUP_DIR/$BACKUP_FILE | cut -f1)"
else
    echo "Backup failed!"
    exit 1
fi

# Delete backups older than 7 days
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +7 -delete
echo "Old backups cleaned up"

echo "Backup complete!"
```

### Script 4: Health Check

```bash
#!/bin/bash

# health_check.sh

check_service() {
    SERVICE=$1
    if systemctl is-active --quiet $SERVICE; then
        echo "✅ $SERVICE is running"
    else
        echo "❌ $SERVICE is NOT running"
    fi
}

check_port() {
    PORT=$1
    if nc -z localhost $PORT 2>/dev/null; then
        echo "✅ Port $PORT is open"
    else
        echo "❌ Port $PORT is closed"
    fi
}

check_disk_space() {
    USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    if [ $USAGE -lt 80 ]; then
        echo "✅ Disk usage: ${USAGE}%"
    else
        echo "⚠️  Disk usage: ${USAGE}% (WARNING)"
    fi
}

check_memory() {
    AVAILABLE=$(free | grep Mem | awk '{print int($7/$2 * 100)}')
    if [ $AVAILABLE -gt 20 ]; then
        echo "✅ Memory available: ${AVAILABLE}%"
    else
        echo "⚠️  Memory available: ${AVAILABLE}% (LOW)"
    fi
}

echo "=== Health Check ==="
echo ""

check_service "nginx"
check_service "docker"
echo ""

check_port 80
check_port 443
check_port 3306
echo ""

check_disk_space
check_memory
```

---

## ৮. Advanced Concepts

### Pipes এবং Redirections:

```bash
# Pipe (|) - output এক command থেকে অন্যটায়
cat file.txt | grep "error" | wc -l

# Redirect output (>)
echo "Hello" > file.txt          # Overwrite
echo "World" >> file.txt         # Append

# Redirect error (2>)
command 2> error.log             # Errors to file
command 2>&1                     # Errors to stdout
command > output.log 2>&1        # Both to file

# Redirect input (<)
wc -l < file.txt

# Discard output
command > /dev/null 2>&1
```

### Command Substitution:

```bash
# Old style
DATE=`date`

# Modern style (preferred)
DATE=$(date)

# Nested
echo "Files: $(ls $(pwd))"

# Arithmetic
RESULT=$((5 + 3))
echo $RESULT  # 8
```

### Arrays:

```bash
# Define array
FRUITS=("apple" "banana" "cherry")

# Access elements
echo ${FRUITS[0]}  # apple
echo ${FRUITS[1]}  # banana

# All elements
echo ${FRUITS[@]}

# Array length
echo ${#FRUITS[@]}  # 3

# Add element
FRUITS+=("orange")

# Loop through array
for FRUIT in "${FRUITS[@]}"; do
    echo $FRUIT
done
```

---

## ৯. Kubernetes Script Examples

### Script 1: Pod Status Check

```bash
#!/bin/bash

# check_pods.sh

NAMESPACE=${1:-default}

echo "Checking pods in namespace: $NAMESPACE"
echo ""

# Get all pods
PODS=$(kubectl get pods -n $NAMESPACE -o name)

for POD in $PODS; do
    POD_NAME=$(echo $POD | cut -d'/' -f2)
    STATUS=$(kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.status.phase}')
    
    if [ "$STATUS" = "Running" ]; then
        echo "✅ $POD_NAME: $STATUS"
    else
        echo "❌ $POD_NAME: $STATUS"
        kubectl describe pod $POD_NAME -n $NAMESPACE | grep -A 5 "Events:"
    fi
done
```

### Script 2: Log Collector

```bash
#!/bin/bash

# collect_logs.sh

NAMESPACE=${1:-default}
OUTPUT_DIR="logs_$(date +%Y%m%d_%H%M%S)"

mkdir -p $OUTPUT_DIR

echo "Collecting logs from namespace: $NAMESPACE"

PODS=$(kubectl get pods -n $NAMESPACE -o jsonpath='{.items[*].metadata.name}')

for POD in $PODS; do
    echo "Collecting logs from $POD..."
    kubectl logs $POD -n $NAMESPACE > "$OUTPUT_DIR/${POD}.log" 2>&1
    
    # If multi-container
    CONTAINERS=$(kubectl get pod $POD -n $NAMESPACE -o jsonpath='{.spec.containers[*].name}')
    for CONTAINER in $CONTAINERS; do
        kubectl logs $POD -c $CONTAINER -n $NAMESPACE > "$OUTPUT_DIR/${POD}_${CONTAINER}.log" 2>&1
    done
done

echo "Logs collected in: $OUTPUT_DIR"
tar -czf "${OUTPUT_DIR}.tar.gz" $OUTPUT_DIR
echo "Archive created: ${OUTPUT_DIR}.tar.gz"
```

---

## ১০. Summary

✅ **Text Processing:**
```bash
grep "pattern" file          # Search
awk '{print $1}' file        # Column processing
sed 's/old/new/g' file       # Replace
```

✅ **File Operations:**
```bash
find /path -name "*.txt"     # Find files
wc -l file                   # Count lines
cut -d':' -f1 /etc/passwd    # Extract column
```

✅ **Network:**
```bash
curl http://example.com      # HTTP client
wget http://file.url         # Download
netstat -tulpn               # Open ports
ss -tulpn                    # Modern netstat
```

✅ **Process:**
```bash
ps aux                       # Process list
kill -9 PID                  # Force kill
top                          # Monitor
```

✅ **Shell Scripting:**
```bash
#!/bin/bash
if [ condition ]; then
    command
fi

for i in {1..10}; do
    echo $i
done
```

এই commands গুলো master করলে Kubernetes debugging অনেক সহজ হবে! 🚀
