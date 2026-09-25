# PID - Process ID

## ১. কেন দরকার? (Why?)

ধরো একটা school এ 1000 জন student আছে। তাদের identify করবে কীভাবে? নামে? কিন্তু দুইজন "Rahul" থাকতে পারে!

**সমাধান: Roll Number** - প্রতিটা student এর unique roll number।

Computer এও same problem:
- একইসাথে 100+ process চলছে
- অনেক process same program (যেমন 5টা `python3`)
- কোনটা কোনটা?

**সমাধান: PID (Process ID)** - প্রতিটা process এর unique number।

### Kubernetes এ কেন দরকার?

- Container এ কোন process crash করছে?
- Process কে signal পাঠাবো কীভাবে?
- Docker container এর main process কোনটা?
- PID namespace কীভাবে container isolate করে?

---

## ২. কী? (What?)

**PID = Process Identifier = একটা unique number যা প্রতিটা running process কে identify করে।**

### Key Points:

```
Process              PID
━━━━━━━              ━━━
systemd        →     1      (Always first)
sshd           →     500
bash           →     1200
python3 app.py →     5678
docker         →     800
nginx          →     9000
```

### PID Characteristics:

1. **Unique:** একই সময়ে দুইটা process এর same PID হবে না
2. **Sequential:** নতুন process কে next available PID দেওয়া হয়
3. **Reusable:** Process শেষ হলে তার PID পরে আবার use করা যাবে
4. **Range:** সাধারণত 1 থেকে 32768 (configurable)

---

## ৩. কীভাবে কাজ করে? (How?)

### PID Allocation:

```
System Boot
     ↓
PID 1: init/systemd     ← প্রথম process
     ↓
PID 2: kthreadd         ← kernel threads
     ↓
PID 3, 4, 5...          ← system processes
     ↓
PID 500+                ← user processes
```

### PID Hierarchy:

```
PID 1 (systemd)
 ├── PID 500 (sshd)
 │   └── PID 1200 (bash)
 │       ├── PID 5678 (python3)
 │       └── PID 5679 (grep)
 └── PID 800 (dockerd)
     └── PID 801 (containerd)
         └── PID 5000 (nginx container)
```

প্রতিটা process এর:
- **PID** - নিজের ID
- **PPID** - Parent Process ID

### PID 1 বিশেষ কেন?

```
PID 1 (init/systemd)
  |
  ├── System initialization
  ├── Orphan process adopt করে
  ├── Shutdown sequence handle করে
  └── Zombie process cleanup করে
```

**Container এ PID 1:**
```
Container
  ├── PID 1: আপনার application (nginx, python, etc.)
  │    ↓
  └── PID 1 exit = container stops!
```

---

## ৪. Hands-On Example

### Example 1: আপনার Shell এর PID

```bash
# Current shell এর PID
echo $$

# Output:
1200

# Parent process এর PID
echo $PPID

# Output:
500    ← sshd অথবা terminal
```

### Example 2: Process PID দেখো

```bash
# সব process এর PID
ps aux

# Output:
USER       PID  %CPU %MEM COMMAND
root         1   0.0  0.1 /sbin/init
apple     1200   0.0  0.0 bash
apple     5678   1.0  2.0 python3 app.py

# Specific process
ps -p 5678

# Parent-child relationship
ps -o pid,ppid,cmd

# Output:
  PID  PPID CMD
 1200   500 bash
 5678  1200 python3 app.py
```

### Example 3: PID দিয়ে Process Find করো

```bash
# PID 5678 কী করছে?
ps -p 5678 -o pid,cmd,%cpu,%mem

# PID 5678 এর সব info
cat /proc/5678/status

# PID 5678 কোন command run করছে?
cat /proc/5678/cmdline
```

### Example 4: New Process এর PID ধরো

```bash
# Background এ run করো
sleep 300 &

# Output:
[1] 9876    ← PID

# Variable এ save করো
MY_PID=$!
echo $MY_PID

# এখন kill করো
kill $MY_PID
```

---

## ৫. PID Namespace (Container Isolation)

### Host vs Container PID:

#### Host Machine:

```bash
ps aux

# Output:
PID   USER     COMMAND
1     root     /sbin/init
800   root     dockerd
5000  root     nginx (container process)
```

#### Container এর ভিতর:

```bash
docker exec <container> ps aux

# Output:
PID   USER     COMMAND
1     root     nginx          ← PID 1!
10    nginx    nginx: worker
```

**Same process, different PID!**

```
Host View:           Container View:
PID 5000     ←→      PID 1

(Global PID)         (Namespace PID)
```

### Diagram:

```
┌─────────────────────────────────────┐
│         Host Machine                │
│                                     │
│  PID 1: systemd                     │
│  PID 800: dockerd                   │
│  PID 5000: container process ────┐  │
│                                  │  │
│  ┌────────────────────────────┐ │  │
│  │  Container (PID Namespace) │ │  │
│  │                            │ │  │
│  │  PID 1: nginx ─────────────┼─┘  │
│  │  PID 10: nginx worker      │    │
│  │                            │    │
│  └────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

## ৬. Docker/Kubernetes এ PID

### Docker Container PID:

```bash
# Container run করো
docker run -d --name mynginx nginx

# Host থেকে container এর PID দেখো
docker inspect mynginx | grep -i pid

# Output:
"Pid": 5000

# Host এ verify করো
ps -p 5000

# Output:
PID   CMD
5000  nginx: master process

# Container এর ভিতর থেকে দেখো
docker exec mynginx ps aux

# Output:
PID   CMD
1     nginx: master process    ← Container এ PID 1
10    nginx: worker process
```

### Kubernetes Pod এ PID:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
# Pod এর process দেখো
kubectl exec mypod -- ps aux

# Output:
PID   USER     COMMAND
1     root     nginx -g daemon off;
10    nginx    nginx: worker process

# Pod এর container ID খুঁজো
kubectl get pod mypod -o jsonpath='{.status.containerStatuses[0].containerID}'

# Node তে SSH করে host PID দেখতে পারো
```

### PID 1 Problem:

#### ❌ Bad Practice:

```dockerfile
# Dockerfile
FROM ubuntu
CMD echo "Hello" && sleep 60
```

```bash
docker run myimage

# Container এর ভিতর:
PID   CMD
1     /bin/sh -c echo Hello && sleep 60    ← Shell as PID 1
10    sleep 60                              ← Actual process
```

**সমস্যা:**
- Shell (PID 1) signals properly forward করে না
- `docker stop` gracefully কাজ করবে না

#### ✅ Good Practice:

```dockerfile
FROM ubuntu
CMD ["sleep", "60"]    # ← Direct command, no shell
```

```bash
# Container এর ভিতর:
PID   CMD
1     sleep 60    ← PID 1 হলো actual process
```

---

## ৭. PID Commands

### Get PID:

```bash
# By name
pidof nginx          # Returns all PIDs
pgrep nginx          # Same

# By pattern
pgrep -f "python.*app.py"

# Full info
pgrep -a python      # PID + command

# Current shell
echo $$

# Last background process
echo $!
```

### Check if PID exists:

```bash
# Method 1: ps
ps -p 5678 > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "Process exists"
fi

# Method 2: kill -0
kill -0 5678 2>/dev/null && echo "Exists"

# Method 3: /proc
[ -d /proc/5678 ] && echo "Exists"
```

### Get Process Info by PID:

```bash
# Basic info
ps -p 5678

# Detailed info
ps -p 5678 -o pid,ppid,user,cmd,%cpu,%mem,etime

# /proc filesystem
cat /proc/5678/status      # Full status
cat /proc/5678/cmdline     # Command line
cat /proc/5678/environ     # Environment variables
ls -l /proc/5678/fd/       # Open file descriptors
cat /proc/5678/maps        # Memory maps
```

---

## ৮. PID Maximum এবং Configuration

### PID Max দেখো:

```bash
# Maximum PID
cat /proc/sys/kernel/pid_max

# Output:
32768
```

### PID Max বাড়াও:

```bash
# Temporary
echo 4194304 | sudo tee /proc/sys/kernel/pid_max

# Permanent
sudo vi /etc/sysctl.conf
# Add:
kernel.pid_max = 4194304

# Apply
sudo sysctl -p
```

### কতগুলো Process চলছে?

```bash
# Current processes
ps aux | wc -l

# Or
ls /proc | grep -E '^[0-9]+$' | wc -l
```

---

## ৯. Practical Exercise

### Task 1: PID Lifecycle

```bash
# 1. একটা process start করো এবং PID save করো
sleep 300 &
MY_PID=$!
echo "Process PID: $MY_PID"

# 2. Process running আছে কিনা check করো
ps -p $MY_PID

# 3. /proc দিয়ে info পড়ো
cat /proc/$MY_PID/status | head -20
cat /proc/$MY_PID/cmdline

# 4. Process kill করো
kill $MY_PID

# 5. আবার check করো (exist করবে না)
ps -p $MY_PID
```

### Task 2: Parent-Child PID

```bash
# 1. একটা shell script লিখো
cat > test.sh << 'EOF'
#!/bin/bash
echo "Parent PID: $$"
echo "Parent PPID: $PPID"

sleep 100 &
CHILD_PID=$!
echo "Child PID: $CHILD_PID"

wait $CHILD_PID
EOF

chmod +x test.sh

# 2. Run করো
./test.sh &

# 3. Process tree দেখো
pstree -p $$
```

### Task 3: Docker PID Namespace

```bash
# 1. Container run করো
docker run -d --name test nginx

# 2. Host PID খুঁজো
HOST_PID=$(docker inspect test --format '{{.State.Pid}}')
echo "Host PID: $HOST_PID"

# 3. Host থেকে process দেখো
ps -p $HOST_PID -o pid,ppid,cmd

# 4. Container এ PID দেখো
docker exec test ps aux

# 5. Compare করো:
echo "Host sees PID: $HOST_PID"
echo "Container sees PID: 1"

# 6. Cleanup
docker stop test && docker rm test
```

### Task 4: PID Monitoring Script

```bash
# একটা script লিখো
cat > monitor_pid.sh << 'EOF'
#!/bin/bash

PID=$1

if [ -z "$PID" ]; then
    echo "Usage: $0 <PID>"
    exit 1
fi

while true; do
    if ps -p $PID > /dev/null 2>&1; then
        CPU=$(ps -p $PID -o %cpu= | tr -d ' ')
        MEM=$(ps -p $PID -o %mem= | tr -d ' ')
        echo "$(date): PID $PID alive - CPU: $CPU% MEM: $MEM%"
    else
        echo "$(date): PID $PID not found!"
        break
    fi
    sleep 2
done
EOF

chmod +x monitor_pid.sh

# Use করো:
# ./monitor_pid.sh 5678
```

---

## ১০. PID Issues এবং Troubleshooting

### Issue 1: PID Exhaustion

```bash
# Free PIDs দেখো
PID_MAX=$(cat /proc/sys/kernel/pid_max)
RUNNING=$(ps aux | wc -l)
FREE=$(($PID_MAX - $RUNNING))

echo "Max PIDs: $PID_MAX"
echo "Running: $RUNNING"
echo "Free: $FREE"

# যদি FREE কম হয়:
# - Zombie processes kill করো
# - Unnecessary processes stop করো
# - PID_MAX বাড়াও
```

### Issue 2: Container PID 1 না Respond করছে

```bash
# Check করো
docker exec mycontainer ps aux

# যদি PID 1 shell হয়:
PID CMD
1   /bin/sh -c ...    ← ❌ Bad
10  actual_app

# Fix: Dockerfile update করো
CMD ["actual_app"]    # No shell wrapper
```

### Issue 3: Process Tracking

```bash
# PID দিয়ে track করা risky:
sleep 100 &
PID=$!

# কিছুক্ষণ পরে process শেষ হলে
# নতুন process same PID পেতে পারে!

# Better: Process name + other identifiers use করো
```

---

## ১১. Interview Questions

### Q1: PID 1 কেন special?

**Answer:**
- Boot এর সময় প্রথম process (init/systemd)
- Orphan processes adopt করে (যাদের parent মরে গেছে)
- Zombie process cleanup করে
- Container এ PID 1 exit করলে container stop হয়
- Signals handle করার responsibility (SIGTERM, SIGCHLD)

### Q2: Container এ PID namespace কীভাবে কাজ করে?

**Answer:**
```
Host:        Container:
PID 5000 ←→  PID 1

- Container নিজের PID namespace এ থাকে
- Container ভাবে সে PID 1 দিয়ে start
- Host সেই same process কে PID 5000 হিসেবে দেখে
- Isolation security provide করে
```

### Q3: Docker stop এ কী signal যায় এবং কোন PID তে?

**Answer:**
```bash
docker stop mycontainer

# Steps:
1. PID 1 তে SIGTERM send করে
2. 10 seconds grace period
3. Still না থামলে SIGKILL

# Graceful shutdown এর জন্য PID 1 কে
# SIGTERM handle করতে হবে
```

### Q4: কীভাবে verify করবে কোন process container এর?

**Answer:**
```bash
# Method 1: Docker inspect
docker inspect <container> --format '{{.State.Pid}}'

# Method 2: /proc filesystem
cat /proc/<PID>/cgroup | grep docker

# Method 3: Docker top
docker top <container>
```

---

## ১২. Summary

✅ **PID (Process ID):**
- প্রতিটা running process এর unique identifier
- 1 থেকে শুরু (init/systemd)
- Process শেষ হলে PID reuse করা যায়

✅ **PID 1:**
- System এর প্রথম process
- Container এ PID 1 exit = container stops
- Orphan ও zombie process handle করে

✅ **PID Namespace:**
- Container আলাদা PID namespace এ থাকে
- Container ভাবে PID 1, host ভাবে অন্য PID
- Isolation provide করে

✅ **Commands:**
```bash
echo $$           # Current shell PID
pidof <name>      # Find PID by name
ps -p <PID>       # Process info
cat /proc/<PID>/  # Detailed info
```

**পরবর্তী পাঠ:** Signals - process কে message পাঠানো! 🚀
