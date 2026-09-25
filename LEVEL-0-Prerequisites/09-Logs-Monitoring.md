# Logs এবং Monitoring - লগ এবং মনিটরিং

## ১. কেন দরকার? (Why?)

ধরো তুমি একটা দোকান চালাও। কিছু জানতে চাও:
- কতজন customer এসেছে? 📊
- কোন product বেশি বিক্রি হয়েছে? 📈
- কোনো সমস্যা হয়েছে কি? ⚠️

**Logs = Application এর diary/notebook**

```
10:00 AM - Customer A entered
10:05 AM - Sold product X
10:10 AM - Error: Payment failed
10:15 AM - Customer B entered
```

Application চলার সময় কী হচ্ছে জানতে logs পড়তে হয়।

### Kubernetes এ কেন দরকার?

- Pod crash করছে কেন? → Logs দেখো
- Application slow কেন? → Logs + metrics দেখো
- কোন error হচ্ছে? → Logs
- Debugging → Logs first!

**No logs = blind troubleshooting! 🙈**

---

## ২. Linux Logs কী এবং কোথায়?

### Log Location:

```
/var/log/
├── syslog          # System general log
├── auth.log        # Authentication (login/sudo)
├── kern.log        # Kernel messages
├── boot.log        # Boot process
├── dmesg           # Kernel ring buffer
├── apt/            # Package manager
│   └── history.log
├── nginx/          # Web server
│   ├── access.log
│   └── error.log
└── mysql/          # Database
    └── error.log
```

### Common Log Files:

| File | কী আছে | কখন দেখবে |
|------|--------|-----------|
| `/var/log/syslog` | System events | General troubleshooting |
| `/var/log/auth.log` | Login attempts, sudo | Security issues |
| `/var/log/kern.log` | Kernel messages | Hardware/driver problems |
| `/var/log/dmesg` | Boot messages | Boot failures |
| `/var/log/nginx/access.log` | HTTP requests | Web traffic analysis |
| `/var/log/nginx/error.log` | Nginx errors | Web server issues |

---

## ৩. Log Formats

### Syslog Format:

```
Sep 25 10:15:32 hostname service[PID]: message

Example:
Sep 25 10:15:32 myserver sshd[1234]: Accepted password for apple from 192.168.1.10
   ↑              ↑        ↑   ↑                    ↑
Timestamp     Hostname  Service PID           Message
```

### Application Logs:

```
[2024-09-25 10:15:32] INFO: User logged in
[2024-09-25 10:15:45] WARNING: Slow query detected
[2024-09-25 10:16:00] ERROR: Database connection failed
```

### Log Levels:

```
DEBUG     → Detailed info (development)
INFO      → General info
WARNING   → Something unusual
ERROR     → Error occurred
CRITICAL  → System failure!
```

---

## ৪. Hands-On: Log Commands

### View Logs:

```bash
# সম্পূর্ণ log file
cat /var/log/syslog

# Last 10 lines
tail /var/log/syslog

# Last 50 lines
tail -n 50 /var/log/syslog

# Real-time monitoring (new lines)
tail -f /var/log/syslog

# First 20 lines
head -n 20 /var/log/syslog
```

### Search Logs:

```bash
# Search for "error"
grep -i error /var/log/syslog

# Search in all logs
grep -r "database" /var/log/

# Case-insensitive, with line numbers
grep -in "failed" /var/log/auth.log

# Show 2 lines before and after
grep -C 2 "error" /var/log/syslog
```

### Filter by Time:

```bash
# Today's logs
grep "Sep 25" /var/log/syslog

# Specific time range
grep "Sep 25 10:1" /var/log/syslog
# Shows: 10:10, 10:11, 10:12, ... 10:19

# Last 1 hour
journalctl --since "1 hour ago"
```

---

## ৫. journalctl (systemd logs)

### journalctl কী?

Modern Linux (systemd based) এ centralized logging:

```
Application/Service
       ↓
    systemd
       ↓
   journald (log daemon)
       ↓
   Binary log storage
       ↓
   journalctl (query tool)
```

### Basic Commands:

```bash
# সব logs
sudo journalctl

# Latest logs (paginated)
sudo journalctl -e

# Follow (real-time)
sudo journalctl -f

# Last 100 lines
sudo journalctl -n 100

# Reverse order (newest first)
sudo journalctl -r
```

### Filter by Service:

```bash
# Specific service
sudo journalctl -u nginx.service
sudo journalctl -u docker.service
sudo journalctl -u kubelet.service

# Multiple services
sudo journalctl -u nginx -u mysql

# Follow service logs
sudo journalctl -u nginx -f
```

### Filter by Time:

```bash
# Since boot
sudo journalctl -b

# Last boot
sudo journalctl -b -1

# Since timestamp
sudo journalctl --since "2024-09-25 10:00:00"

# Until timestamp
sudo journalctl --until "2024-09-25 11:00:00"

# Time range
sudo journalctl --since "10:00" --until "11:00"

# Relative time
sudo journalctl --since "1 hour ago"
sudo journalctl --since "30 minutes ago"
sudo journalctl --since "yesterday"
```

### Filter by Priority:

```bash
# Priority levels:
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug

# Errors and above
sudo journalctl -p err

# Warnings and above
sudo journalctl -p warning

# Specific service + priority
sudo journalctl -u nginx -p err
```

### Follow Logs:

```bash
# Real-time, all logs
sudo journalctl -f

# Specific service
sudo journalctl -u nginx -f

# With priority filter
sudo journalctl -u nginx -p err -f

# Last 50 lines + follow
sudo journalctl -n 50 -f
```

---

## ৬. Application Logs

### Python Logging:

```python
import logging

# Configure
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

# Use
logger.debug("Debug message")
logger.info("User logged in")
logger.warning("Memory usage high")
logger.error("Database connection failed")
logger.critical("System shutdown!")
```

```bash
# View logs
tail -f app.log

# Output:
2024-09-25 10:15:32,123 [INFO] User logged in
2024-09-25 10:16:45,456 [WARNING] Memory usage high
2024-09-25 10:17:00,789 [ERROR] Database connection failed
```

### Node.js Logging:

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

logger.info('User logged in');
logger.warn('High memory usage');
logger.error('Database error');
```

---

## ৭. Docker Logs

### Container Logs:

```bash
# Container logs দেখো
docker logs mycontainer

# Last 100 lines
docker logs --tail 100 mycontainer

# Follow (real-time)
docker logs -f mycontainer

# With timestamps
docker logs -t mycontainer

# Since specific time
docker logs --since "2024-09-25T10:00:00" mycontainer

# Last 1 hour
docker logs --since 1h mycontainer
```

### Where are Docker Logs?

```bash
# Host machine এ:
/var/lib/docker/containers/<container-id>/<container-id>-json.log

# Example:
cat /var/lib/docker/containers/abc123.../abc123...-json.log

# Format: JSON
{"log":"Hello World\n","stream":"stdout","time":"2024-09-25T10:15:32.123Z"}
```

### Docker Logging Drivers:

```bash
# Check current driver
docker info | grep "Logging Driver"

# Common drivers:
# - json-file (default)
# - syslog
# - journald
# - none

# Run with specific driver
docker run --log-driver=syslog myimage
docker run --log-driver=journald myimage

# No logs (performance)
docker run --log-driver=none myimage
```

### Docker Compose Logs:

```bash
# All services
docker-compose logs

# Specific service
docker-compose logs app

# Follow
docker-compose logs -f

# Tail
docker-compose logs --tail=100

# Multiple services
docker-compose logs app db
```

---

## ৮. Kubernetes Logs

### Pod Logs:

```bash
# Single container Pod
kubectl logs mypod

# Multi-container Pod (specify container)
kubectl logs mypod -c mycontainer

# Follow
kubectl logs -f mypod

# Previous container (if crashed)
kubectl logs mypod --previous

# Last 100 lines
kubectl logs mypod --tail=100

# Since time
kubectl logs mypod --since=1h
kubectl logs mypod --since-time="2024-09-25T10:00:00Z"
```

### Deployment Logs:

```bash
# সব Pods এর logs
kubectl logs deployment/myapp

# Follow all Pods
kubectl logs -f deployment/myapp

# Specific replica
kubectl logs deployment/myapp --tail=100
```

### Label Selector:

```bash
# Label দিয়ে filter
kubectl logs -l app=myapp

# Multiple labels
kubectl logs -l app=myapp,env=production

# Follow
kubectl logs -f -l app=myapp
```

### Where are Kubernetes Logs?

```
Node:
/var/log/pods/<namespace>_<pod-name>_<pod-uid>/<container-name>/

Example:
/var/log/pods/default_mypod_abc-123-xyz/mycontainer/0.log
```

```bash
# Node তে SSH করে
ls /var/log/pods/
cat /var/log/pods/default_mypod_*/mycontainer/0.log
```

---

## ৯. Monitoring Commands

### System Resources:

#### top - Real-time Process Monitoring:

```bash
top

# Output:
top - 10:15:32 up 5 days, 2:30, 2 users, load average: 0.5, 0.4, 0.3
Tasks: 150 total, 1 running, 149 sleeping
%Cpu(s): 5.2 us, 2.1 sy, 0.0 ni, 92.5 id
MiB Mem: 8000 total, 6000 used, 2000 free
MiB Swap: 2000 total, 0 used, 2000 free

PID  USER  PR  NI  VIRT   RES   SHR S %CPU %MEM TIME+    COMMAND
1234 apple 20  0   500M   200M  50M  S  5.2  2.5  10:30.50 python3
5678 root  20  0   100M   50M   20M  S  2.1  0.6  5:15.30  nginx

# Keys:
# P - Sort by CPU
# M - Sort by Memory
# k - Kill process
# q - Quit
# 1 - Show all CPUs
```

#### htop - Better top:

```bash
htop

# Features:
# - Colorful
# - Mouse support
# - Easy navigation
# - Tree view (F5)
```

#### ps - Process Snapshot:

```bash
# All processes
ps aux

# Specific user
ps -u apple

# Process tree
ps auxf

# Specific columns
ps -eo pid,ppid,cmd,%cpu,%mem

# Sort by CPU
ps aux --sort=-%cpu | head -10

# Sort by memory
ps aux --sort=-%mem | head -10
```

### Memory:

```bash
# Memory usage
free -h

# Output:
              total   used   free   shared  buff/cache  available
Mem:          7.8Gi  3.2Gi  1.5Gi  200Mi   3.1Gi       4.2Gi
Swap:         2.0Gi  0B     2.0Gi

# Detailed memory
cat /proc/meminfo
```

### Disk:

```bash
# Disk usage
df -h

# Output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   60G   40G  60% /
/dev/sda2       500G  200G  300G  40% /data

# Directory size
du -sh /var/log
du -sh /home/*

# Largest directories
du -sh /* | sort -h
```

### Network:

```bash
# Network interfaces
ip addr show

# Routing table
ip route show

# Network statistics
netstat -i

# Active connections
netstat -tulpn

# Or (modern)
ss -tulpn

# Bandwidth usage (need iftop)
sudo iftop
```

### Load Average:

```bash
# System load
uptime

# Output:
10:15:32 up 5 days, 2:30, 2 users, load average: 0.5, 0.4, 0.3
                                                    ↑    ↑    ↑
                                                   1min 5min 15min

# Load average meaning:
# < 1.0  - System idle
# 1.0    - Full capacity
# > 1.0  - Overloaded (processes waiting)
```

---

## ১০. Practical Exercise

### Task 1: Log Analysis

```bash
# 1. View system log
tail -n 50 /var/log/syslog

# 2. Search for errors
grep -i error /var/log/syslog | tail -20

# 3. Auth log (login attempts)
sudo grep "Failed password" /var/log/auth.log

# 4. Successful logins
sudo grep "Accepted password" /var/log/auth.log

# 5. Real-time monitoring
tail -f /var/log/syslog
```

### Task 2: journalctl Practice

```bash
# 1. Service logs
sudo journalctl -u docker.service -n 50

# 2. Errors only
sudo journalctl -p err -n 20

# 3. Last 1 hour
sudo journalctl --since "1 hour ago"

# 4. Follow specific service
sudo journalctl -u nginx -f

# 5. Boot logs
sudo journalctl -b | grep -i error
```

### Task 3: Docker Logs

```bash
# 1. Run nginx container
docker run -d --name testnginx nginx

# 2. Generate some logs (access it)
curl http://localhost:80

# 3. View logs
docker logs testnginx

# 4. Follow logs
docker logs -f testnginx

# 5. Tail last 10
docker logs --tail 10 testnginx

# 6. Cleanup
docker stop testnginx
docker rm testnginx
```

### Task 4: Kubernetes Logs

```bash
# 1. Create a Pod
kubectl run testpod --image=nginx

# 2. Check logs
kubectl logs testpod

# 3. Generate logs (exec into pod)
kubectl exec testpod -- ls /

# 4. View new logs
kubectl logs testpod --tail=20

# 5. Follow
kubectl logs -f testpod

# 6. Cleanup
kubectl delete pod testpod
```

### Task 5: System Monitoring

```bash
# 1. CPU/Memory usage
top

# 2. Specific process
ps aux | grep nginx

# 3. Memory details
free -h

# 4. Disk usage
df -h
du -sh /var/*

# 5. Network connections
sudo netstat -tulpn
# Or
sudo ss -tulpn

# 6. System load
uptime
```

---

## ১১. Log Rotation

### কেন দরকার?

```
Day 1: app.log (10 MB)
Day 2: app.log (20 MB)
Day 3: app.log (30 MB)
...
Day 30: app.log (300 MB) 😱

Disk full!
```

**Solution: Log Rotation** - পুরোনো logs archive/delete করো।

### logrotate Configuration:

```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                # Daily rotation
    rotate 7             # Keep 7 days
    compress             # Compress old logs
    delaycompress        # Don't compress latest
    missingok            # OK if log missing
    notifempty           # Don't rotate if empty
    create 0644 root root # New file permissions
}
```

```bash
# Test
sudo logrotate -d /etc/logrotate.d/myapp

# Force rotate
sudo logrotate -f /etc/logrotate.d/myapp

# Result:
# app.log        (current)
# app.log.1      (yesterday, not compressed)
# app.log.2.gz   (2 days ago, compressed)
# app.log.3.gz   (3 days ago)
# ...
```

---

## ১২. Centralized Logging (Production)

### Problem:

```
Server 1: app.log
Server 2: app.log
Server 3: app.log
...
Server 50: app.log

😱 কোনটা দেখবো?
```

### Solution: Centralized Logging

```
Application Servers (50)
      ↓
   Log Shipper (Fluentd/Filebeat)
      ↓
   Log Storage (Elasticsearch)
      ↓
   Visualization (Kibana)
```

### ELK Stack:

- **E**lasticsearch - Log storage & search
- **L**ogstash - Log processing
- **K**ibana - Visualization

### Kubernetes Logging:

```
Pods → Node logs → Log Collector (Fluentd) → Elasticsearch → Kibana
```

---

## ১৩. Common Issues

### Issue 1: Logs Too Large

```bash
# Check size
du -sh /var/log/*

# Clean old logs
sudo find /var/log -name "*.gz" -mtime +30 -delete

# Setup logrotate
```

### Issue 2: No Logs Showing

```bash
# Check if service writing logs
sudo journalctl -u myservice

# Check permissions
ls -l /var/log/myapp.log

# Check log driver (Docker)
docker inspect mycontainer | grep LogPath
```

### Issue 3: Disk Full from Logs

```bash
# Find large logs
sudo du -sh /var/log/* | sort -h

# Emergency cleanup
sudo journalctl --vacuum-time=2d
sudo journalctl --vacuum-size=500M

# Docker logs cleanup
docker system prune -a
```

---

## ১৪. Interview Questions

### Q1: Kubernetes Pod crash করছে, কীভাবে debug করবে?

**Answer:**
```bash
# 1. Status check
kubectl get pods

# 2. Details
kubectl describe pod mypod

# 3. Current logs
kubectl logs mypod

# 4. Previous container logs (if crashed)
kubectl logs mypod --previous

# 5. Events
kubectl get events --sort-by='.lastTimestamp'
```

### Q2: journalctl vs /var/log files?

**Answer:**

| | journalctl | /var/log/syslog |
|---|---|---|
| **Storage** | Binary | Text |
| **Query** | Fast, indexed | grep/search |
| **Filter** | Built-in filters | Manual tools |
| **Rotation** | Automatic | logrotate |
| **Persistence** | Configurable | Always persisted |

### Q3: Docker logs কোথায় stored?

**Answer:**
```bash
# Location:
/var/lib/docker/containers/<container-id>/<container-id>-json.log

# Driver dependent:
# json-file: stored in file
# syslog: sent to syslog
# journald: sent to journald
# none: no storage
```

---

## ১৫. Summary

✅ **Logs Location:**
```
/var/log/syslog        # System
/var/log/auth.log      # Authentication
journalctl             # systemd logs
```

✅ **View Logs:**
```bash
tail -f /var/log/syslog
grep -i error /var/log/syslog
journalctl -u service -f
```

✅ **Docker:**
```bash
docker logs mycontainer
docker logs -f --tail 100 mycontainer
```

✅ **Kubernetes:**
```bash
kubectl logs mypod
kubectl logs -f mypod -c container
kubectl logs mypod --previous
```

✅ **Monitoring:**
```bash
top, htop           # Process monitoring
free -h             # Memory
df -h               # Disk
netstat -tulpn      # Network
```

**পরবর্তী পাঠ:** Essential Linux commands এবং shell scripting! 🚀
