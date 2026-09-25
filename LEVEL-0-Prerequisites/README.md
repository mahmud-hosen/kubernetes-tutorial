# LEVEL 0 - Prerequisites

Kubernetes শেখার আগে এই foundational topics গুলো জানা অত্যন্ত জরুরি। প্রতিটা tutorial বাংলায় লেখা এবং step-by-step examples সহ।

## 📚 Topics Covered

### 1. [Linux Filesystem](01-Linux-Filesystem.md)
- **কেন দরকার:** File organization, system structure বোঝা
- **কী শিখবে:**
  - Linux directory structure (`/`, `/etc`, `/var`, `/home`)
  - Absolute vs Relative paths
  - `/proc` এবং `/sys` virtual filesystems
  - Container filesystem isolation
  - Volume mounting concepts

**Kubernetes Connection:** Pod volumes, ConfigMap mounting, log locations

---

### 2. [Process](02-Process.md)
- **কেন দরকার:** Running applications কীভাবে কাজ করে
- **কী শিখবে:**
  - Process কী এবং Program এর সাথে পার্থক্য
  - Process lifecycle (fork, exec, exit)
  - Process states (Running, Sleeping, Zombie)
  - Parent-child relationship
  - Background vs Foreground processes

**Kubernetes Connection:** Container = Process, PID 1 importance, CrashLoopBackOff debugging

---

### 3. [PID - Process ID](03-PID.md)
- **কেন দরকার:** Process identify এবং manage করা
- **কী শিখবে:**
  - PID কী এবং কীভাবে allocate হয়
  - PID 1 এর বিশেষত্ব
  - PID namespace (container isolation)
  - Host PID vs Container PID
  - Process tracking এবং monitoring

**Kubernetes Connection:** Container PID namespace, init process, signal handling

---

### 4. [Signals](04-Signals.md)
- **কেন দরকার:** Process এর সাথে communicate করা
- **কী শিখবে:**
  - Signal কী (SIGTERM, SIGKILL, SIGINT)
  - Graceful shutdown vs Force kill
  - Signal handlers
  - Trap command (Bash)
  - Python/Node.js signal handling

**Kubernetes Connection:** Pod termination, graceful shutdown, preStop hooks

---

### 5. [Environment Variables](05-Environment-Variables.md)
- **কেন দরকার:** Configuration management
- **কী শিখবে:**
  - Environment variables কী
  - PATH variable
  - Docker এ ENV
  - ConfigMap/Secret থেকে ENV inject
  - 12-factor app pattern

**Kubernetes Connection:** ConfigMap, Secret, env/envFrom, container configuration

---

### 6. [Networking Basics](06-Networking-Basics.md)
- **কেন দরকার:** Network communication বোঝা
- **কী শিখবে:**
  - IP Address (IPv4, private/public)
  - Localhost (127.0.0.1)
  - Ports এবং port mapping
  - TCP vs UDP
  - DNS resolution
  - HTTP/HTTPS request-response
  - Docker networking basics

**Kubernetes Connection:** Pod IP, Service, Ingress, DNS, ClusterIP

---

### 7. [Docker Complete Guide](07-Docker-Complete-Guide.md)
- **কেন দরকার:** Kubernetes Pod = Docker containers
- **কী শিখবে:**
  - Image vs Container
  - Dockerfile লেখা
  - Docker commands (run, build, exec, logs)
  - Volumes (data persistence)
  - Networks (container-to-container)
  - Docker Compose
  - Image layers এবং caching
  - Docker Registry

**Kubernetes Connection:** Container images, Pod containers, volume mounts, multi-container Pods

---

### 8. [Users, Groups & Permissions](08-Users-Groups-Permissions.md)
- **কেন দরকার:** Security এবং access control
- **কী শিখবে:**
  - Linux users এবং groups
  - File permissions (rwx, 644, 755)
  - chmod, chown commands
  - Root vs non-root users
  - SecurityContext in containers
  - setuid, setgid, sticky bit

**Kubernetes Connection:** runAsUser, runAsNonRoot, fsGroup, SecurityContext, Pod Security Standards

---

### 9. [Logs & Monitoring](09-Logs-Monitoring.md)
- **কেন দরকার:** Troubleshooting এবং debugging
- **কী শিখবে:**
  - Linux log locations (`/var/log`)
  - journalctl (systemd logs)
  - Docker logs
  - Kubernetes Pod logs
  - Monitoring tools (top, htop, ps)
  - Resource monitoring (CPU, memory, disk)
  - Log rotation

**Kubernetes Connection:** kubectl logs, container logs, events, metrics-server

---

### 10. [Essential Linux Commands & Shell Scripting](10-Essential-Linux-Commands-Shell-Scripting.md)
- **কেন দরকার:** Day-to-day troubleshooting
- **কী শিখবে:**
  - Text processing (grep, awk, sed)
  - File operations (find, wc, cut)
  - Network tools (curl, wget, netstat, ss)
  - Process management (ps, kill, top)
  - Shell scripting basics
  - Conditionals, loops, functions
  - Practical automation scripts

**Kubernetes Connection:** Pod debugging, log parsing, health checks, automation scripts

---

## 🎯 Learning Path

```
1. Start → Linux Filesystem
2. → Process & PID
3. → Signals
4. → Environment Variables
5. → Networking Basics
6. → Docker Complete Guide ← Most Critical!
7. → Users & Permissions
8. → Logs & Monitoring
9. → Linux Commands & Scripting
10. Ready for Kubernetes! ✅
```

---

## 📖 কীভাবে পড়বে?

### প্রতিটা Tutorial এর Structure:

1. **কেন দরকার?** - Motivation এবং real-world context
2. **কী?** - Concept explanation
3. **কীভাবে কাজ করে?** - Internal mechanics
4. **Hands-on Examples** - Practical commands
5. **Docker/Kubernetes Relationship** - কীভাবে এটা Kubernetes এ use হয়
6. **Practical Exercise** - নিজে try করো
7. **Troubleshooting** - Common issues
8. **Interview Questions** - Important Q&A
9. **Summary** - Quick recap

---

## ✅ Prerequisites Checklist

শুরু করার আগে নিশ্চিত করো:

- [ ] Linux machine access আছে (Ubuntu/Debian recommended)
- [ ] Terminal/Shell use করতে পারো
- [ ] Basic typing এবং text editor (nano/vim) জানো
- [ ] Docker installed আছে (or install করতে পারবে)
- [ ] Internet connection (packages download এর জন্য)

---

## 🛠️ Setup Instructions

### Ubuntu/Debian:

```bash
# System update
sudo apt update && sudo apt upgrade -y

# Essential tools install
sudo apt install -y curl wget git vim net-tools

# Docker install
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Verify
docker --version
```

### macOS:

```bash
# Homebrew install (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Docker Desktop install
# Download from: https://www.docker.com/products/docker-desktop

# Verify
docker --version
```

---

## 🧪 Test Your Knowledge

প্রতিটা tutorial শেষে এই questions নিজেকে জিজ্ঞাসা করো:

1. **Can I explain this concept to someone else?**
2. **Have I tried all the hands-on examples?**
3. **Do I understand how this relates to Kubernetes?**
4. **Can I solve the practice exercises without looking?**

যদি উত্তর "না" হয়, তাহলে সেই section আবার পড়ো!

---

## 🔗 Quick Reference

### Most Important Commands:

```bash
# Files & Directories
ls -la, cd, pwd, mkdir, rm, cp, mv, find

# Text Processing  
cat, grep, awk, sed, less, head, tail

# Process Management
ps aux, top, kill, pkill, pgrep

# System Info
df -h, free -h, uptime, uname -a

# Networking
curl, wget, netstat, ss, ping, ip addr

# Docker
docker run, docker ps, docker logs, docker exec, docker build

# Logs
tail -f, journalctl, grep

# Permissions
chmod, chown, ls -l
```

---

## 📝 Notes

- সব commands নিজের machine এ try করো
- Error আসলে ঘাবড়াবে না - debugging শেখার part!
- প্রতিটা concept এর Kubernetes connection ভালো করে বুঝো
- Practical exercises skip করো না

---

## ⏭️ Next Steps

LEVEL 0 complete করার পর:

1. সব concepts review করো
2. Practice exercises আবার করো
3. [LEVEL 1 - Kubernetes Fundamentals](../LEVEL-1-Kubernetes-Fundamentals/) এ যাও

---

## 🎓 Why These Topics?

### Docker → Kubernetes Connection:

```
Docker Concept          Kubernetes Equivalent
─────────────          ─────────────────────
docker run             kubectl run / Pod
docker-compose         Deployment + Service
-v (volume)            PersistentVolumeClaim
-e (env)               ConfigMap / Secret
-p (port)              Service
--network              Service / NetworkPolicy
docker logs            kubectl logs
docker exec            kubectl exec
```

### Linux → Kubernetes Connection:

```
Linux Concept          Kubernetes Usage
─────────────          ─────────────────
Process                Container (PID 1)
Signals                Pod termination
Environment Vars       ConfigMap / Secret
Users/Permissions      SecurityContext
Logs                   kubectl logs / Events
Filesystem             Volumes / ConfigMaps
Networking             Pod networking / Service
```

---

## 💡 Pro Tips

1. **Muscle Memory:** Commands practice করো daily
2. **Man Pages:** `man <command>` পড়ো
3. **Errors:** Error messages ভালো করে পড়ো - এগুলো helpful!
4. **Google:** "How to X in Linux/Docker" search করতে শেখো
5. **Documentation:** Official docs সবসময় best resource

---

## 🚀 Ready?

Prerequisites complete করার পর তুমি ready থাকবে:

✅ Container concepts বুঝতে
✅ Kubernetes architecture বুঝতে
✅ Pod troubleshoot করতে
✅ Logs analyze করতে
✅ Networking debug করতে
✅ Security best practices বুঝতে

**শুভ শিক্ষা! Happy Learning! 🎉**

---

## 📞 Help & Support

যদি কোনো confusion হয়:

1. Tutorial আবার পড়ো (ধৈর্য রাখো!)
2. Hands-on examples নিজে try করো
3. Error messages Google করো
4. Linux/Docker documentation পড়ো

**Remember: Every expert was once a beginner! 💪**
