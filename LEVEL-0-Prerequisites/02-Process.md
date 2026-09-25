# Process - প্রসেস

## ১. কেন দরকার? (Why?)

ধরো তুমি একটা restaurant এ আছো। সেখানে:
- Cook খাবার বানাচ্ছে
- Waiter serve করছে
- Cashier bill করছে

একইসময়ে অনেকগুলো কাজ চলছে। কম্পিউটারেও একইসাথে অনেক কিছু চলে:
- Browser run হচ্ছে
- Music player চলছে
- Background update হচ্ছে

**Process হলো এই প্রতিটা running program।**

### Kubernetes এ কেন দরকার?

- একটা Pod এর ভিতরে process run হয়
- Container আসলে একটা process
- Process crash হলে container restart হয়
- Process থামালে container থেমে যায়

Process না বুঝলে container troubleshoot করতে পারবে না!

---

## ২. কী? (What?)

**Process** = একটা running program এর instance।

### Program vs Process:

**Program:**
- File যা disk এ থাকে
- উদাহরণ: `/usr/bin/python3`
- Static, চলছে না

**Process:**
- Program যা memory তে load হয়ে চলছে
- উদাহরণ: `python3 app.py` run করলে একটা process তৈরি হয়
- Dynamic, CPU use করছে

### একটা Program থেকে অনেক Process:

```bash
# তুমি 3 বার Firefox খুললে:
firefox &    # Process 1
firefox &    # Process 2
firefox &    # Process 3

# তিনটা আলাদা process, same program
```

---

## ৩. কীভাবে কাজ করে? (How?)

### Process Lifecycle:

```
Program
   ↓
  fork()      ← নতুন process তৈরি
   ↓
  exec()      ← Program load করো
   ↓
Running      ← CPU execute করছে
   ↓
 Exit        ← Process শেষ
   ↓
 Zombie      ← Parent cleanup করেনি (optional)
```

### Process এর Components:

```
Process
├── PID (Process ID)                 ← Unique number
├── Parent PID (PPID)                ← কে create করেছে
├── Memory Space                     ← RAM এর একটা part
│   ├── Code
│   ├── Data
│   ├── Stack
│   └── Heap
├── File Descriptors                 ← Open files
│   ├── stdin (0)
│   ├── stdout (1)
│   └── stderr (2)
├── CPU Time                         ← কতক্ষণ CPU use করেছে
└── State                            ← Running/Sleeping/Stopped
```

### Process States:

| State | Symbol | মানে |
|-------|--------|------|
| Running | R | CPU execute করছে |
| Sleeping | S | কিছুর জন্য wait করছে (disk, network) |
| Stopped | T | Stopped (Ctrl+Z) |
| Zombie | Z | শেষ হয়ে গেছে, parent cleanup করেনি |

---

## ৪. Hands-On Example

### Example 1: Process List দেখো

```bash
# সব process দেখো
ps aux

# Output:
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 168820 13452 ?        Ss   10:00   0:01 /sbin/init
apple     1234  2.5  1.0 450000 80000 ?        Sl   10:05   0:30 /usr/bin/python3
```

**Column মানে:**
- `PID` - Process ID
- `%CPU` - CPU usage
- `%MEM` - Memory usage
- `VSZ` - Virtual memory size
- `RSS` - Physical memory (RAM)
- `STAT` - State (R/S/T/Z)
- `COMMAND` - কোন program

### Example 2: একটা Process Start করো

```bash
# একটা Python script run করো
python3 -c "import time; time.sleep(60)" &

# Output:
[1] 5678   ← PID

# Process টা দেখো
ps aux | grep 5678
```

### Example 3: Process Tree দেখো

```bash
# Parent-child relationship দেখো
pstree -p

# Output:
systemd(1)─┬─sshd(500)───sshd(1200)───bash(1201)───pstree(5000)
           ├─dockerd(800)───containerd(801)
           └─kubelet(900)
```

এখানে:
- `systemd` (PID 1) সবার parent
- `sshd` এর child হলো `bash`
- `bash` এর child হলো `pstree`

### Example 4: Process Monitor করো (Real-time)

```bash
# top command (interactive)
top

# Output:
PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
1234 apple    20   0  450000  80000  20000 S   2.5   1.0   0:30.00 python3
5678 root     20   0  200000  40000  10000 R  50.0   0.5   5:00.00 nginx
```

**Keys:**
- `q` - Quit
- `k` - Kill process
- `P` - Sort by CPU
- `M` - Sort by Memory
- `1` - Show all CPUs

---

## ৫. Process Creation - fork() এবং exec()

### Step by Step:

```c
// Parent process
int pid = fork();    // ← New process তৈরি (exact copy)

if (pid == 0) {
    // Child process
    exec("/bin/ls");  // ← নতুন program load করো
} else {
    // Parent process
    wait(NULL);       // ← Child শেষ হওয়া পর্যন্ত wait
}
```

### Diagram:

```
Parent Process (PID 100)
       |
       | fork()
       |
       ↓
Child Process (PID 101) [exact copy of parent]
       |
       | exec("/bin/ls")
       |
       ↓
Now running: ls (PID 101)
```

### Bash Example:

```bash
# Bash process
$ python3 app.py    ← Bash creates a child process

# Bash process তৈরি করলো child:
# 1. fork() - Copy of bash
# 2. exec() - Replace with python3
```

---

## ৬. Docker/Kubernetes Relationship

### Docker Container = Process!

```bash
# Container run করো
docker run -d nginx

# Container এর main process দেখো
docker top <container-id>

# Output:
UID     PID    PPID   CMD
root    1234   1233   nginx: master process
```

### Key Points:

#### 1. Container এর PID 1:

```bash
# Container এর ভিতর থেকে
docker exec -it <container> bash
ps aux

# Output:
PID   USER     COMMAND
  1   root     nginx          ← Main process
 10   root     nginx: worker
```

PID 1 শেষ হলে container মরে যায়!

#### 2. Host Machine এর PID:

```bash
# Host থেকে দেখো
ps aux | grep nginx

# Output:
PID   USER     COMMAND
5678  root     nginx    ← Container এর PID 1
                          ← Host এ আলাদা PID!
```

Container এর PID 1 আর host এর PID আলাদা! (Namespace isolation)

### Kubernetes Pod এ Process:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["nginx"]           # ← এই process PID 1 হবে
    args: ["-g", "daemon off;"]
```

```bash
# Pod এর process দেখো
kubectl exec mypod -- ps aux

# Output:
PID   USER     COMMAND
  1   root     nginx -g daemon off;   ← Main process
 10   nginx    nginx: worker process
```

### CrashLoopBackOff = Process Crash:

```bash
# Pod status
kubectl get pods

# Output:
NAME    READY   STATUS             RESTARTS
mypod   0/1     CrashLoopBackOff   5

# কেন crash হচ্ছে? Process immediately exit করছে!
kubectl logs mypod

# সমাধান: Process যেন চলতে থাকে ensure করো
```

---

## ৭. Process Management Commands

### View Processes:

```bash
# সব process
ps aux

# শুধু তোমার process
ps -u $USER

# Full hierarchy
ps auxf

# Specific process
ps -p 1234

# By name
ps aux | grep nginx
```

### Process Information:

```bash
# Detailed info
ps -p 1234 -o pid,ppid,cmd,%cpu,%mem

# Process tree
pstree -p 1234

# Open files
lsof -p 1234

# Number of threads
ps -T -p 1234
```

### Filter Process:

```bash
# Find process by name
pgrep nginx         # Returns PIDs
pgrep -a nginx      # Returns PIDs + command

# Kill by name
pkill nginx
```

---

## ৮. Foreground vs Background Process

### Foreground Process:

```bash
# Terminal block করে রাখে
python3 app.py

# Ctrl+C দিয়ে stop করতে হয়
```

### Background Process:

```bash
# Background এ run করো
python3 app.py &

# Output:
[1] 5678    ← Job number এবং PID

# Terminal free, অন্য কাজ করতে পারো
```

### Jobs Management:

```bash
# Running jobs
jobs

# Output:
[1]+  Running    python3 app.py &

# Foreground এ আনো
fg %1

# Background এ পাঠাও (আগে Ctrl+Z চাপো)
Ctrl+Z
bg %1
```

### Nohup (No Hangup):

```bash
# Terminal close করলেও চলবে
nohup python3 app.py &

# Output: nohup.out file এ যাবে
```

---

## ৯. Practical Exercise

### Task 1: Process Lifecycle

```bash
# 1. একটা long-running process start করো
sleep 300 &
echo $!    # PID save করো

# 2. Process টা running আছে কিনা check করো
ps -p <PID>

# 3. Process এর parent দেখো
ps -o pid,ppid,cmd -p <PID>

# 4. Process kill করো
kill <PID>

# 5. Zombie আছে কিনা check করো
ps aux | grep 'Z'
```

### Task 2: Docker Process

```bash
# 1. Nginx container run করো
docker run -d --name mynginx nginx

# 2. Container এর main process PID দেখো (container এর ভিতর)
docker exec mynginx ps aux

# 3. Host machine এ same process এর PID দেখো
docker inspect mynginx | grep Pid

# 4. Host থেকে process দেখো
ps -p <HOST_PID>

# 5. Container এর PID 1 kill করো
docker exec mynginx kill 1
# Container restart হবে!
```

### Task 3: Process Monitoring

```bash
# 1. top চালাও এবং highest CPU usage process খুঁজো
top

# 2. একটা Python script লিখো যা CPU use করবে
cat > cpu_test.py << 'EOF'
while True:
    x = 1 + 1
EOF

python3 cpu_test.py &
PID=$!

# 3. top এ এই process দেখো
top -p $PID

# 4. Kill করো
kill $PID
```

---

## ১০. Process vs Thread

### Process:

```
Process 1             Process 2
├── Memory            ├── Memory
├── File Descriptors  ├── File Descriptors
└── CPU Time          └── CPU Time

  (Isolated)             (Isolated)
```

### Thread:

```
Process
├── Memory (Shared)
├── File Descriptors (Shared)
└── Threads
    ├── Thread 1 (own CPU state)
    ├── Thread 2 (own CPU state)
    └── Thread 3 (own CPU state)
```

**Example:**

```bash
# Chrome browser:
# - 1 main process
# - প্রতিটা tab একটা আলাদা process

# Python threading:
# - 1 process
# - একাধিক thread (same memory)
```

---

## ১১. Common Issues

### Issue 1: Too Many Processes

```bash
# Process limit দেখো
ulimit -u

# Output: 4096

# Current running processes
ps aux | wc -l

# Fork bomb (NEVER RUN!)
# :(){ :|:& };:
# এটা infinite processes create করবে, system hang!
```

### Issue 2: Zombie Process

```bash
# Zombie খুঁজো
ps aux | awk '$8 ~ /Z/ {print}'

# Zombie কেন?
# Parent process child এর exit status collect করেনি (wait())

# Fix: Parent process কে kill করো
kill <PPID>
```

### Issue 3: Container Immediately Exits

```yaml
# ❌ Bad
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: test
    image: ubuntu
    command: ["echo", "hello"]  # ← Process immediately exit!

# ✅ Good
  - name: test
    image: ubuntu
    command: ["sleep", "3600"]  # ← Process keeps running
```

---

## ১২. Interview Questions

### Q1: PID 1 কেন বিশেষ?

**Answer:**
- System এর প্রথম process (init/systemd)
- অন্য সব process এর parent (directly or indirectly)
- Container এ PID 1 exit করলে container মরে যায়
- Signal handling বিশেষ (SIGTERM/SIGKILL properly handle করা উচিত)

### Q2: Docker container কেন একটা process?

**Answer:**
- Container run করলে একটা main process start হয়
- সেই process চলার সময় container চলবে
- Process শেষ হলে container stopped অবস্থায় যাবে
- Container isolation namespace দিয়ে করা (PID namespace)

### Q3: Zombie process কীভাবে identify করবে?

**Answer:**
```bash
# State দেখো
ps aux | awk '$8 ~ /Z/'

# Or
top    # Status column এ 'Z'
```

Zombie হয় যখন child process শেষ কিন্তু parent এখনো `wait()` করেনি।

### Q4: কীভাবে Kubernetes Pod এর process debug করবে?

**Answer:**
```bash
# 1. Running process দেখো
kubectl exec mypod -- ps aux

# 2. PID 1 কী?
kubectl exec mypod -- ps -p 1

# 3. Log দেখো
kubectl logs mypod

# 4. Process crash করছে কিনা
kubectl describe pod mypod
# Events section check করো
```

---

## ১৩. Summary

✅ **Process:**
- Running program এর instance
- প্রতিটা process এর unique PID আছে
- Parent-child relationship আছে

✅ **Process States:**
- Running (R) - CPU use করছে
- Sleeping (S) - Wait করছে
- Zombie (Z) - শেষ কিন্তু cleanup হয়নি

✅ **Docker/Kubernetes:**
- Container = Process (isolated)
- PID 1 exit করলে container মরে যায়
- Container crash = process crash

✅ **Commands:**
- `ps aux` - Process list
- `top` - Real-time monitoring
- `pgrep/pkill` - Find/kill by name
- `docker top` - Container processes

**পরবর্তী পাঠ:** PID (Process ID) এবং PID Namespace! 🚀
