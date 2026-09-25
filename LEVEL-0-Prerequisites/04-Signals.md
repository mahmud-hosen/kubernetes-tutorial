# Signals - সিগন্যাল

## ১. কেন দরকার? (Why?)

ধরো তুমি একটা restaurant এ waiter। Kitchen থেকে তোমাকে message পাঠানোর দরকার:
- 🔔 "Order ready!"
- ⏰ "Close করার time!"  
- 🚨 "Emergency - এখনই বন্ধ করো!"

Running process এর সাথে communicate করার জন্য Operating System **Signals** ব্যবহার করে।

### Real-Life Examples:

```bash
# Ctrl+C চাপলে
→ SIGINT signal → Process বন্ধ হয়

# Terminal close করলে
→ SIGHUP signal → Process terminate

# kill command
→ SIGTERM/SIGKILL signal → Process stop
```

### Kubernetes এ কেন দরকার?

- Pod delete করলে container কে SIGTERM signal যায়
- Graceful shutdown implement করতে signal handle দরকার
- Container crash = SIGKILL
- Health check fail = process কে signal

Signal না বুঝলে Pod gracefully shutdown হবে না!

---

## ২. কী? (What?)

**Signal = একটা software interrupt যা process কে কিছু ঘটেছে বা ঘটতে যাচ্ছে জানায়।**

এটা একটা **asynchronous notification** - যেকোনো সময় আসতে পারে।

### মনে রাখো:

```
Signal = Message to Process

কে পাঠায়:
- Operating System
- অন্য Process  
- নিজেই নিজেকে

কেন পাঠায়:
- Process বন্ধ করতে
- Reload করতে
- Error জানাতে
- Debug করতে
```

---

## ৩. Common Signals

### Major Signals List:

| Signal | Number | মানে | Default Action | Catchable? |
|--------|--------|------|----------------|-----------|
| SIGHUP | 1 | Hangup (terminal closed) | Terminate | ✅ Yes |
| SIGINT | 2 | Interrupt (Ctrl+C) | Terminate | ✅ Yes |
| SIGQUIT | 3 | Quit (Ctrl+\\) | Core dump | ✅ Yes |
| SIGKILL | 9 | Kill (force) | Terminate | ❌ No |
| SIGTERM | 15 | Terminate (graceful) | Terminate | ✅ Yes |
| SIGSTOP | 19 | Stop (pause) | Stop | ❌ No |
| SIGCONT | 18 | Continue | Resume | ✅ Yes |
| SIGCHLD | 17 | Child status changed | Ignore | ✅ Yes |

### Signal Categories:

```
📍 Termination Signals:
   SIGTERM (15) - Graceful shutdown
   SIGKILL (9)  - Force kill
   SIGINT (2)   - Interrupt

📍 Job Control:
   SIGSTOP (19) - Pause
   SIGCONT (18) - Resume
   SIGTSTP (20) - Ctrl+Z

📍 Error Signals:
   SIGSEGV (11) - Segmentation fault
   SIGFPE (8)   - Floating point error
   SIGILL (4)   - Illegal instruction

📍 Special:
   SIGCHLD (17) - Child process ended
   SIGHUP (1)   - Terminal closed
```

---

## ৪. কীভাবে কাজ করে? (How?)

### Signal Flow:

```
Sender                     Receiver (Process)
  |                              |
  | kill -15 1234                |
  |                              |
  |-------- SIGTERM ------------>|
                                 |
                            1. Interrupt করে
                            2. Signal handler চালায়
                            3. Default অথবা custom action
```

### Signal Handling:

Process একটা signal পেলে তিনটা option:

```
Signal পেলাম
    |
    ├─→ Default action (terminate/ignore/stop)
    ├─→ Custom handler (আমার code চালাও)
    └─→ Ignore (কিছু করো না)
```

### Example: Python Signal Handler

```python
import signal
import time

def signal_handler(signum, frame):
    print(f"Signal {signum} received! Gracefully shutting down...")
    # Cleanup code
    exit(0)

# SIGTERM handle করো
signal.signal(signal.SIGTERM, signal_handler)

# SIGINT (Ctrl+C) handle করো  
signal.signal(signal.SIGINT, signal_handler)

print("Running... Press Ctrl+C to stop")
while True:
    time.sleep(1)
```

```bash
# Run করো
python3 app.py

# Test করো
kill -SIGTERM <PID>    # Graceful shutdown
# অথবা Ctrl+C          # SIGINT
```

---

## ৫. Hands-On Example

### Example 1: Signal পাঠাও

```bash
# একটা long process start করো
sleep 300 &
PID=$!

# SIGTERM পাঠাও (graceful)
kill -15 $PID
# অথবা
kill -SIGTERM $PID
# অথবা simply
kill $PID         # Default SIGTERM

# Verify (process থাকবে না)
ps -p $PID
```

### Example 2: SIGKILL (Force Kill)

```bash
# একটা process যা SIGTERM ignore করে
cat > ignore_sigterm.sh << 'EOF'
#!/bin/bash
trap '' SIGTERM  # SIGTERM ignore করো
echo "PID: $$"
while true; do
    sleep 1
done
EOF

chmod +x ignore_sigterm.sh
./ignore_sigterm.sh &
PID=$!

# SIGTERM পাঠাও (কাজ করবে না)
kill -SIGTERM $PID
ps -p $PID    # এখনো চলছে!

# SIGKILL পাঠাও (force, cannot be ignored)
kill -9 $PID
ps -p $PID    # এখন মরে গেছে
```

### Example 3: Job Control (STOP/CONT)

```bash
# একটা process start করো
sleep 300 &
PID=$!

# STOP signal (pause করো)
kill -STOP $PID

# Check state
ps -p $PID -o pid,state,cmd
# STATE: T (Stopped)

# CONT signal (resume করো)
kill -CONT $PID

# Check again
ps -p $PID -o pid,state,cmd  
# STATE: S (Sleeping) বা R (Running)
```

### Example 4: Ctrl+C এবং Ctrl+Z

```bash
# একটা command run করো
sleep 300

# Ctrl+C চাপো
# → SIGINT signal → Process terminate

# আবার run করো
sleep 300

# Ctrl+Z চাপো  
# → SIGTSTP signal → Process stopped
[1]+  Stopped   sleep 300

# Background এ চালাও
bg %1

# Foreground এ আনো
fg %1

# এখন Ctrl+C দিয়ে stop করো
```

---

## ৬. Docker/Kubernetes এ Signals

### Docker Container Lifecycle:

```bash
docker stop <container>
```

**কী হয়:**

```
1. Docker PID 1 তে SIGTERM পাঠায়
   ↓
2. 10 seconds wait (grace period)
   ↓
3. যদি এখনো চলে, তাহলে SIGKILL
   ↓
4. Container stopped
```

### Graceful Shutdown Example:

```python
# app.py
import signal
import sys
import time

def cleanup():
    print("Closing database connections...")
    print("Flushing logs...")
    print("Goodbye!")
    sys.exit(0)

def signal_handler(sig, frame):
    print(f"Received signal {sig}")
    cleanup()

signal.signal(signal.SIGTERM, signal_handler)
signal.signal(signal.SIGINT, signal_handler)

print("Application started")
while True:
    print("Working...")
    time.sleep(5)
```

```dockerfile
FROM python:3.9
COPY app.py /app.py
CMD ["python3", "/app.py"]
```

```bash
# Build এবং run
docker build -t myapp .
docker run --name test myapp

# অন্য terminal থেকে stop (graceful)
docker stop test

# Logs দেখো
docker logs test
# "Received signal 15"
# "Closing database connections..."
```

### Kubernetes Pod Termination:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  terminationGracePeriodSeconds: 30  # ← Grace period
  containers:
  - name: app
    image: myapp
```

**Pod delete করলে:**

```
kubectl delete pod mypod
    ↓
1. Pod "Terminating" state
    ↓
2. Container PID 1 তে SIGTERM
    ↓
3. Wait up to 30 seconds
    ↓
4. যদি এখনো চলে → SIGKILL
    ↓
5. Pod deleted
```

### PreStop Hook:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: app
    image: myapp
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 5"]
```

**Lifecycle:**

```
Pod delete request
    ↓
1. preStop hook চালায়
    ↓  
2. SIGTERM পাঠায়
    ↓
3. Grace period wait
    ↓
4. SIGKILL (if needed)
```

---

## ৭. Signal Commands

### Send Signal:

```bash
# By PID
kill -SIGTERM 1234
kill -15 1234
kill 1234              # Default: SIGTERM

# Force kill
kill -9 1234
kill -SIGKILL 1234

# By name
pkill -SIGTERM nginx
killall -9 python3

# Current process group
kill -SIGTERM 0
```

### List Signals:

```bash
# All signals
kill -l

# Output:
1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
5) SIGTRAP      6) SIGABRT      7) SIGBUS       8) SIGFPE
9) SIGKILL     10) SIGUSR1     11) SIGSEGV     12) SIGUSR2
...

# Signal number খুঁজো
kill -l SIGTERM
# Output: 15

# Number থেকে name
kill -l 15  
# Output: TERM
```

### Monitor Signals:

```bash
# Process কী signals handle করে?
cat /proc/<PID>/status | grep Sig

# Output:
SigQ:   0/15391
SigPnd: 0000000000000000  # Pending signals
SigBlk: 0000000000000000  # Blocked signals
SigIgn: 0000000000001000  # Ignored signals
SigCgt: 0000000180000000  # Caught signals
```

---

## ৮. Signal Handling - Trap Command (Bash)

### Example Script:

```bash
#!/bin/bash

# Cleanup function
cleanup() {
    echo "Caught signal! Cleaning up..."
    # Remove temp files
    rm -f /tmp/myapp.lock
    exit 0
}

# Trap signals
trap cleanup SIGTERM SIGINT

echo "Script running with PID $$"
echo "Creating lock file..."
touch /tmp/myapp.lock

# Simulate work
while true; do
    echo "Working..."
    sleep 2
done
```

```bash
# Run করো
./script.sh &
PID=$!

# Signal পাঠাও
kill -SIGTERM $PID

# Output দেখবে:
# "Caught signal! Cleaning up..."

# Lock file cleaned আছে কিনা check করো
ls /tmp/myapp.lock    # File নেই!
```

### Advanced Trap:

```bash
#!/bin/bash

# Multiple handlers
handle_sigterm() {
    echo "SIGTERM received"
    exit 0
}

handle_sigint() {
    echo "SIGINT received (Ctrl+C)"
    exit 0
}

handle_sighup() {
    echo "SIGHUP received - reloading config"
    # Reload configuration
}

trap handle_sigterm SIGTERM
trap handle_sigint SIGINT  
trap handle_sighup SIGHUP

while true; do
    sleep 1
done
```

---

## ৯. Practical Exercise

### Task 1: Signal Testing

```bash
# 1. একটা test script লিখো
cat > signal_test.sh << 'EOF'
#!/bin/bash

cleanup() {
    echo "Cleanup function called"
    exit 0
}

trap cleanup SIGTERM SIGINT

echo "PID: $$"
echo "Try: kill -SIGTERM $$"

while true; do
    echo "Running..."
    sleep 2
done
EOF

chmod +x signal_test.sh

# 2. Run করো (background)
./signal_test.sh &
PID=$!

# 3. Different signals try করো
kill -SIGTERM $PID    # Graceful
kill -SIGINT $PID     # Ctrl+C equivalent
kill -9 $PID          # Force (যদি আগেরগুলো কাজ না করে)
```

### Task 2: Docker Signal Handling

```bash
# 1. একটা Python app লিখো
cat > app.py << 'EOF'
import signal
import time
import sys

def handler(sig, frame):
    print(f"Signal {sig} received")
    print("Shutting down gracefully...")
    time.sleep(2)  # Simulate cleanup
    print("Done!")
    sys.exit(0)

signal.signal(signal.SIGTERM, handler)
signal.signal(signal.SIGINT, handler)

print("App started")
while True:
    print("Working...")
    time.sleep(3)
EOF

# 2. Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.9-slim
COPY app.py /app.py
CMD ["python3", "/app.py"]
EOF

# 3. Build এবং test
docker build -t sigtest .
docker run --name test sigtest

# অন্য terminal:
docker stop test        # Graceful (10s grace period)
docker logs test        # Check output

# Force kill test
docker run --name test2 sigtest
docker kill test2       # Immediate SIGKILL
docker logs test2       # Handler চালায়নি
```

### Task 3: Kubernetes Graceful Shutdown

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sigtest
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: app
    image: sigtest
    lifecycle:
      preStop:
        exec:
          command: ["sh", "-c", "echo 'PreStop hook' && sleep 5"]
```

```bash
# Deploy
kubectl apply -f pod.yaml

# Watch logs
kubectl logs -f sigtest

# Delete (অন্য terminal)
kubectl delete pod sigtest

# Logs এ দেখবে:
# 1. "PreStop hook"
# 2. "Signal 15 received"
# 3. "Shutting down gracefully..."
```

---

## ১০. Common Issues

### Issue 1: SIGTERM Ignored

```bash
# Process SIGTERM handle করছে না
docker stop mycontainer
# 10 seconds wait → তারপর SIGKILL

# Debug:
docker exec mycontainer ps aux
# PID 1 কী? Shell হলে signal forward নাও করতে পারে

# Fix: Proper entrypoint
CMD ["python3", "app.py"]    # Not: CMD python3 app.py
```

### Issue 2: Zombie Processes

```bash
# Parent process SIGCHLD handle করে না
# → Child exit করলেও cleanup হয় না
# → Zombie accumulate হয়

# Check:
ps aux | grep 'Z'

# Fix: Parent এ proper signal handling
```

### Issue 3: Wrong Grace Period

```yaml
# ❌ Too short
terminationGracePeriodSeconds: 5
# Application cleanup করার আগেই SIGKILL

# ✅ Appropriate
terminationGracePeriodSeconds: 30
# Enough time for graceful shutdown
```

---

## ১১. Interview Questions

### Q1: SIGTERM এবং SIGKILL এর পার্থক্য কী?

**Answer:**

| | SIGTERM (15) | SIGKILL (9) |
|---|---|---|
| **Catchable** | Yes | No |
| **Graceful** | Yes | No |
| **Use case** | Normal shutdown | Force kill |
| **Cleanup** | Allows | Doesn't allow |

```bash
# SIGTERM: Process handle করতে পারে
kill -15 1234

# SIGKILL: Cannot be caught/ignored
kill -9 1234
```

### Q2: `docker stop` এবং `docker kill` এর পার্থক্য?

**Answer:**

```bash
docker stop:
1. SIGTERM পাঠায়
2. 10s grace period
3. Then SIGKILL

docker kill:
1. Immediately SIGKILL
2. No grace period
```

### Q3: Kubernetes Pod gracefully shutdown কীভাবে?

**Answer:**

```yaml
terminationGracePeriodSeconds: 30
lifecycle:
  preStop:
    exec:
      command: ["/cleanup.sh"]
```

```
1. preStop hook
2. SIGTERM to PID 1  
3. Wait grace period
4. SIGKILL if still running
```

### Q4: Signal handle করার benefit কী?

**Answer:**

✅ **Graceful shutdown:**
- Database connections close করো
- Ongoing requests complete করো
- Logs flush করো
- Temp files clean করো

❌ **No handling:**
- Abrupt termination
- Data loss
- Corrupted state
- Resource leaks

---

## ১২. Summary

✅ **Signals:**
- Process কে message পাঠানোর mechanism
- SIGTERM (15) - Graceful shutdown
- SIGKILL (9) - Force kill (uncatchable)
- SIGINT (2) - Ctrl+C

✅ **Signal Handling:**
```python
import signal
signal.signal(signal.SIGTERM, handler)
```

```bash
trap cleanup SIGTERM SIGINT
```

✅ **Docker/Kubernetes:**
- `docker stop` → SIGTERM → wait → SIGKILL
- Pod delete → preStop → SIGTERM → grace period → SIGKILL
- PID 1 must handle signals properly

✅ **Commands:**
```bash
kill -15 <PID>        # SIGTERM
kill -9 <PID>         # SIGKILL
kill -l               # List signals
trap handler SIGTERM  # Bash handler
```

**পরবর্তী পাঠ:** systemd - Linux service management! 🚀
