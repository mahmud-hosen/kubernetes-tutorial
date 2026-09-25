# Linux Filesystem - লিনাক্স ফাইলসিস্টেম

## ১. কেন দরকার? (Why?)

ধরো তুমি একটা বাড়িতে থাকো। সেই বাড়িতে রান্নাঘর, বেডরুম, বাথরুম আলাদা আলাদা জায়গায় থাকে। কেন? কারণ সব জিনিস এক জায়গায় রাখলে খুঁজে পাওয়া কঠিন হয়ে যায়।

**Linux Filesystem** ঠিক একই কারণে দরকার:
- কম্পিউটারে লক্ষ লক্ষ file থাকে
- এগুলো organized না রাখলে কিছু খুঁজে পাওয়া অসম্ভব
- প্রতিটি file একটা নির্দিষ্ট জায়গায় থাকলে system তা দ্রুত খুঁজে পায়
- সিকিউরিটির জন্য: কিছু file শুধু admin দেখবে, কিছু সবাই দেখবে

**Kubernetes-এ দরকার কেন?**
- Container-এর ভিতরে একটা mini Linux filesystem থাকে
- Application logs কোথায় রাখা হবে?
- Configuration file কোথায় পড়বে?
- Data কোথায় save করবে?
এই সব জানতে হলে Linux filesystem বুঝতে হবে।

---

## ২. কী? (What?)

**Linux Filesystem** হলো Linux-এ file এবং directory গুলো কীভাবে organize করা হয় তার একটা hierarchical structure (গাছের মতো গঠন)।

### Windows vs Linux

**Windows:**
```
C:\
D:\
E:\
```
প্রতিটা drive আলাদা।

**Linux:**
```
/
```
শুধু একটা root `/` থেকে সব শুরু। সব file এবং directory এই `/` এর নিচে।

---

## ৩. কীভাবে কাজ করে? (How?)

### Linux Filesystem Hierarchy:

```
/                          ← Root (সবকিছুর শুরু)
├── bin/                   ← Essential commands (ls, cat, cp)
├── boot/                  ← Bootloader files
├── dev/                   ← Device files (হার্ডডিস্ক, USB)
├── etc/                   ← Configuration files
├── home/                  ← User directories
│   ├── user1/
│   └── user2/
├── lib/                   ← Libraries
├── media/                 ← Removable media (USB mount)
├── mnt/                   ← Temporary mount
├── opt/                   ← Optional software
├── proc/                  ← Process information (virtual)
├── root/                  ← Root user home
├── run/                   ← Runtime data
├── sbin/                  ← System binaries (admin commands)
├── srv/                   ← Service data
├── sys/                   ← System information (virtual)
├── tmp/                   ← Temporary files (সবাই লিখতে পারে)
├── usr/                   ← User programs
│   ├── bin/
│   ├── lib/
│   └── local/
└── var/                   ← Variable data (logs, cache)
    ├── log/               ← Log files
    ├── cache/
    └── tmp/
```

### গুরুত্বপূর্ণ Directory গুলো:

| Directory | কী থাকে | উদাহরণ |
|-----------|---------|---------|
| `/bin` | Basic commands | `ls`, `cat`, `cp`, `mv` |
| `/etc` | Configuration | `/etc/passwd`, `/etc/nginx/nginx.conf` |
| `/home` | User files | `/home/johndoe/documents/` |
| `/var/log` | Log files | `/var/log/nginx/access.log` |
| `/tmp` | Temporary files | Delete হয় reboot এ |
| `/proc` | Running process info | `/proc/1234/` (PID 1234) |
| `/dev` | Device files | `/dev/sda1` (disk), `/dev/null` |

---

## ৪. Hands-On Example

### Example 1: Filesystem Explore করো

```bash
# Root directory দেখো
ls /

# Output:
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

```bash
# etc directory তে কী আছে?
ls /etc/

# কিছু important config দেখো
ls /etc/passwd      # User list
ls /etc/hostname    # Computer name
ls /etc/hosts       # DNS mapping
```

### Example 2: আপনার Home Directory

```bash
# তুমি কোথায় আছো?
pwd
# Output: /home/apple

# তোমার home এ কী আছে?
ls ~
# ~ মানে home directory
```

### Example 3: Path বুঝো (Absolute vs Relative)

**Absolute Path** (পুরো address):
```bash
/home/apple/Documents/file.txt
```
সবসময় `/` দিয়ে শুরু।

**Relative Path** (বর্তমান location থেকে):
```bash
# ধরো তুমি /home/apple তে আছো
cd Documents          # relative path
cd /home/apple/Documents  # absolute path (same)
```

### Example 4: File Create করো Different Directories তে

```bash
# Temporary file (reboot এ delete হবে)
touch /tmp/test.txt
ls /tmp/test.txt

# তোমার home এ file
touch ~/myfile.txt
ls ~/myfile.txt

# এখন try করো /etc তে file রাখতে
touch /etc/test.txt
# ❌ Permission denied!
# কেন? /etc শুধু root user modify করতে পারে
```

---

## ৫. Docker/Kubernetes Relationship

### Docker Container এর Filesystem:

যখন তুমি একটা Docker container run করো:

```bash
docker run -it ubuntu bash
```

Container এর ভিতরে পুরো Linux filesystem আছে:

```bash
root@container:/# ls /
bin  boot  dev  etc  home  lib  ...
```

কিন্তু এটা **isolated**! Container এর `/etc` আর তোমার host machine এর `/etc` আলাদা।

### Kubernetes তে Filesystem:

**Pod এর মধ্যে:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d/   # ← Filesystem path!
  volumes:
  - name: config
    configMap:
      name: nginx-config
```

এখানে:
- `/etc/nginx/conf.d/` হলো container এর ভিতরের filesystem path
- Kubernetes এই path এ config file mount করবে

**Log files:**
```bash
# Container এর log কোথায়?
kubectl logs mypod

# আসলে Kubernetes পড়ছে:
# /var/log/pods/<namespace>_<pod>_<uid>/...
```

### Volume Mount Example:

```yaml
volumes:
- name: app-data
  hostPath:
    path: /mnt/data           # ← Host machine এর path
    
volumeMounts:
- name: app-data
  mountPath: /app/data        # ← Container এর path
```

Container `/app/data` তে write করলে তা host এর `/mnt/data` তে save হবে।

---

## ৬. Common Filesystem Commands

### Navigation:

```bash
pwd                    # বর্তমান directory
ls                     # List files
ls -la                 # Details সহ
cd /path/to/dir        # Change directory
cd ..                  # এক level উপরে
cd ~                   # Home directory
cd -                   # আগের directory
```

### File Operations:

```bash
touch file.txt         # Empty file create
mkdir mydir            # Directory create
cp file.txt backup.txt # Copy
mv file.txt newname.txt # Move/Rename
rm file.txt            # Delete file
rm -rf mydir/          # Delete directory (সাবধান!)
```

### File Information:

```bash
file myfile.txt        # File type
stat myfile.txt        # Detailed info
du -sh /var/log        # Directory size
df -h                  # Disk usage
```

### Finding Files:

```bash
# Find by name
find /home -name "*.txt"

# Find by type
find /var -type f      # Files
find /var -type d      # Directories

# Find by size
find /tmp -size +100M  # > 100MB
```

---

## ৭. /proc এবং /sys (Special Filesystems)

এগুলো **virtual filesystems** - আসলে disk এ নেই, kernel memory থেকে আসে।

### /proc Example:

```bash
# Process 1234 এর info
ls /proc/1234/
cat /proc/1234/cmdline    # Command যা দিয়ে run করা হয়েছে
cat /proc/1234/status     # Process status

# System info
cat /proc/cpuinfo         # CPU info
cat /proc/meminfo         # Memory info
cat /proc/uptime          # System uptime
```

### Kubernetes এ /proc:

```bash
# Pod এর ভিতরে
kubectl exec -it mypod -- bash
cat /proc/1/cmdline       # Container এর main process
```

---

## ৮. Practical Exercise

### Task 1: Filesystem Explore

```bash
# 1. Root directory তে কয়টা folder আছে?
ls / | wc -l

# 2. /etc তে nginx config আছে কিনা check করো
ls /etc/nginx/

# 3. তোমার home directory এর size কত?
du -sh ~

# 4. /tmp তে একটা file create করো
touch /tmp/mytest-$(date +%s).txt

# 5. System এ কত memory free আছে?
free -h
```

### Task 2: Path Practice

```bash
# 1. Absolute path দিয়ে home যাও
cd /home/$USER

# 2. Relative path দিয়ে /tmp যাও
cd ../../tmp

# 3. আবার home ফিরে এসো shortcut দিয়ে
cd ~

# 4. তোমার current directory এর absolute path দেখাও
pwd
```

### Task 3: Docker Filesystem

```bash
# 1. একটা Ubuntu container run করো
docker run -it --rm ubuntu bash

# 2. Container এর ভিতর থেকে run করো:
ls /
ls /etc/
cat /etc/os-release
df -h

# 3. একটা file create করো
echo "Hello from container" > /tmp/test.txt

# 4. Exit করো এবং আবার নতুন container run করো
exit
docker run -it --rm ubuntu bash
ls /tmp/test.txt    # ❌ File নেই! কেন? Container isolated
```

---

## ৯. Common Mistakes এবং Tips

### ❌ Mistake 1: Root Directory Delete করা

```bash
# NEVER DO THIS!
rm -rf /    # পুরো system destroy হবে
```

### ❌ Mistake 2: /tmp কে Permanent Storage ভাবা

```bash
# ❌ Bad:
echo "important data" > /tmp/data.txt
# Reboot এ delete হয়ে যাবে

# ✅ Good:
echo "important data" > ~/data.txt
```

### ✅ Tip 1: Disk Full হলে কী করবে?

```bash
# কোন directory সবচেয়ে বেশি space নিচ্ছে?
du -sh /* | sort -h

# Log files clean করো
sudo rm /var/log/*.log
```

### ✅ Tip 2: Hidden Files

```bash
ls        # Hidden files দেখায় না
ls -a     # Hidden files দেখায় (.bashrc, .ssh)
```

---

## ১০. Interview Questions

### Q1: `/bin` এবং `/usr/bin` এর মধ্যে পার্থক্য কী?

**Answer:**
- `/bin` - Essential commands যা boot time এ দরকার (ls, cat, cp)
- `/usr/bin` - User programs (python, git, node)
- Emergency mode এ `/usr` mount না থাকলেও `/bin` থাকে

### Q2: `/tmp` এবং `/var/tmp` এর পার্থক্য?

**Answer:**
- `/tmp` - Reboot এ clear হয়
- `/var/tmp` - Reboot এর পরেও থাকে (but may be cleaned periodically)

### Q3: Container filesystem কি host filesystem থেকে আলাদা?

**Answer:**
হ্যাঁ! Container এর নিজস্ব filesystem layer থাকে:
- Base image layer (read-only)
- Container layer (writable)
- Container delete হলে writable layer মুছে যায়
- Volume mount করলে host filesystem এর সাথে data share করা যায়

### Q4: Kubernetes Pod filesystem কোথায় থাকে?

**Answer:**
Host node এর `/var/lib/kubelet/pods/<pod-uid>/` তে।

---

## ১১. Summary

✅ **Linux Filesystem:**
- `/` থেকে সব শুরু (hierarchical)
- প্রতিটা directory এর নির্দিষ্ট purpose আছে
- `/etc` - config, `/var/log` - logs, `/tmp` - temporary

✅ **Path:**
- Absolute path: `/home/user/file.txt`
- Relative path: `./file.txt`, `../parent/`

✅ **Virtual Filesystems:**
- `/proc` - Process info
- `/sys` - Hardware info

✅ **Docker/Kubernetes:**
- Container এর নিজস্ব isolated filesystem
- Volume mount করে host filesystem share করা যায়
- Log files `/var/log/pods/` তে থাকে

**পরবর্তী পাঠ:** Process এবং PID বুঝবো! 🚀
