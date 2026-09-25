# Users, Groups & Permissions - ইউজার, গ্রুপ এবং অনুমতি

## ১. কেন দরকার? (Why?)

ধরো একটা অফিস:
- সবাই সব ফাইল দেখতে পারবে না
- Manager HR ফাইল দেখবে, developer না
- Read-only access vs write access

Linux-ও একই system!

```
User: "এই file delete করতে দাও"
Linux: "Permission denied! তোমার access নেই।"
```

### Kubernetes এ কেন দরকার?

- Container কোন user হিসেবে চলবে? Root নাকি non-root?
- File permissions না বুঝলে container crash করবে
- Security best practice: run as non-root user
- RBAC (Role-Based Access Control) এর base concept

---

## ২. Users কী? (What is User?)

### Linux User = একজন ব্যক্তি বা process যে system access করতে পারে

```bash
# Current user কে?
whoami
# Output: apple

# User ID (UID)
id
# Output: uid=1000(apple) gid=1000(apple) groups=1000(apple),27(sudo)
```

### User Types:

#### 1. Root User (Superuser):

```
Username: root
UID: 0
Powers: সবকিছু করতে পারে! 🔥
```

```bash
# Root হিসেবে command চালাও
sudo command

# Root user এ switch করো
sudo su -

# এখন prompt:
root@hostname:~#
```

#### 2. Normal User:

```
Username: apple, john, etc.
UID: 1000+
Powers: সীমিত (নিজের files, specific directories)
```

#### 3. System User:

```
Username: www-data, mysql, nginx
UID: 1-999
Purpose: Services চালানোর জন্য
```

```bash
# System user দেখো
cat /etc/passwd | grep -E "www-data|mysql|nginx"

# www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
#   ↑      ↑ ↑  ↑     ↑        ↑          ↑
# name   pwd UID GID comment home       shell
```

---

## ৩. Groups কী?

### Group = User এর collection

একটা user multiple groups এ থাকতে পারে।

```
Group: developers
  ├── alice
  ├── bob
  └── charlie

Group: admins
  ├── alice  ← alice দুই group এ!
  └── david
```

### Why Groups?

```bash
# ❌ Without groups:
# প্রতিটা user কে individually permission দিতে হবে

# ✅ With groups:
# একবার group কে permission, সব users পাবে!
```

### Primary vs Secondary Groups:

```bash
# User info
id apple

# Output:
uid=1000(apple) gid=1000(apple) groups=1000(apple),27(sudo),999(docker)
#                   ↑                    ↑           ↑         ↑
#                Primary              Primary    Secondary  Secondary
```

- **Primary group:** User create file করলে এই group owner হবে
- **Secondary groups:** Additional access

---

## ৪. Permissions কী?

### File Permissions:

```bash
ls -l myfile.txt

# Output:
-rw-r--r-- 1 apple developers 1024 Sep 25 10:00 myfile.txt
│││ │ │ │      ↑       ↑        ↑       ↑          ↑
│││ │ │ │    link   owner   group    size       name
│││ │ │ │    count
│││ │ │ └─ others
│││ │ └─── group
│││ └───── owner
││└────── file type
│└─────── permissions
└──────── file type (- = regular file)
```

### Permission Breakdown:

```
-rw-r--r--
│││ │ │ │
│││ │ │ └─ Others: r-- (read only)
│││ │ └─── Group:  r-- (read only)
│││ └───── Owner:  rw- (read + write)
││└─────── File type
│└──────── Permission string
└───────── File type: - (regular file)
```

### Permission Types:

| Symbol | Meaning | On File | On Directory |
|--------|---------|---------|--------------|
| `r` | Read | পড়তে পারবে | List করতে পারবে (ls) |
| `w` | Write | লিখতে/modify করতে পারবে | File create/delete করতে পারবে |
| `x` | Execute | Run করতে পারবে | ভিতরে ঢুকতে পারবে (cd) |

### Permission Examples:

```bash
-rw-r--r--   # Regular file, owner rw, others r
drwxr-xr-x   # Directory, owner rwx, others rx
-rwxr-xr-x   # Executable, all can read+execute
-rw-------   # Owner only, very private
drwxrwxrwx   # Everyone can do anything (dangerous!)
```

---

## ৫. Numeric Permissions (chmod)

### Octal Numbers:

```
Binary    Decimal    Permission
─────     ────────   ──────────
0 0 0        0       ---
0 0 1        1       --x
0 1 0        2       -w-
0 1 1        3       -wx
1 0 0        4       r--
1 0 1        5       r-x
1 1 0        6       rw-
1 1 1        7       rwx
```

### Common Permission Codes:

```
644 = rw-r--r--   (files)
  ↑   ↑  ↑  ↑
  6   4  4
Owner Group Others

755 = rwxr-xr-x   (directories, executables)
  ↑   ↑   ↑   ↑
  7   5   5
Owner Group Others

600 = rw-------   (private files, SSH keys)
700 = rwx------   (private directories)
777 = rwxrwxrwx   (❌ dangerous! everyone)
```

---

## ৬. Hands-On Examples

### Example 1: Create User

```bash
# নতুন user create (root দরকার)
sudo useradd -m -s /bin/bash john
# -m: home directory তৈরি করো
# -s: shell set করো

# Password set করো
sudo passwd john

# User info দেখো
id john
grep john /etc/passwd

# Switch to that user
su - john
```

### Example 2: Create Group

```bash
# Group তৈরি করো
sudo groupadd developers

# User কে group এ add করো
sudo usermod -aG developers john
# -a: append (existing groups keep করো)
# -G: secondary group

# Verify
groups john
# Output: john : john developers
```

### Example 3: File Permissions

```bash
# একটা file তৈরি করো
echo "Hello" > myfile.txt

# Permissions দেখো
ls -l myfile.txt
# -rw-r--r-- 1 apple apple 6 Sep 25 10:00 myfile.txt

# Owner কে execute permission দাও
chmod u+x myfile.txt
# u: user (owner)
# +: add
# x: execute

ls -l myfile.txt
# -rwxr--r-- 1 apple apple 6 Sep 25 10:00 myfile.txt

# Group কে write permission দাও
chmod g+w myfile.txt
ls -l myfile.txt
# -rwxrw-r-- 1 apple apple 6 Sep 25 10:00 myfile.txt

# Others থেকে read remove করো
chmod o-r myfile.txt
ls -l myfile.txt
# -rwxrw---- 1 apple apple 6 Sep 25 10:00 myfile.txt
```

### Example 4: Numeric Permissions

```bash
# 644: rw-r--r--
chmod 644 myfile.txt
ls -l myfile.txt
# -rw-r--r-- 1 apple apple 6 Sep 25 10:00 myfile.txt

# 755: rwxr-xr-x
chmod 755 myscript.sh
ls -l myscript.sh
# -rwxr-xr-x 1 apple apple 100 Sep 25 10:00 myscript.sh

# 600: rw------- (private)
chmod 600 id_rsa
ls -l id_rsa
# -rw------- 1 apple apple 1234 Sep 25 10:00 id_rsa
```

### Example 5: Change Owner

```bash
# Owner change করো (root দরকার)
sudo chown john myfile.txt
ls -l myfile.txt
# -rw-r--r-- 1 john apple 6 Sep 25 10:00 myfile.txt
#                ↑
#              Owner changed!

# Group change করো
sudo chgrp developers myfile.txt
ls -l myfile.txt
# -rw-r--r-- 1 john developers 6 Sep 25 10:00 myfile.txt
#                      ↑
#                  Group changed!

# Owner + group একসাথে
sudo chown john:developers myfile.txt

# Recursive (directory এর সব files)
sudo chown -R john:developers /path/to/directory
```

---

## ৭. Special Permissions

### Setuid (Set User ID):

```bash
# Setuid bit set করো
chmod u+s /usr/bin/passwd

ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ...
#     ↑
#   's' instead of 'x' = setuid

# যেকোনো user চালালে root হিসেবে execute হবে!
```

**Example:** `passwd` command:
- User নিজের password change করে
- কিন্তু `/etc/shadow` file শুধু root modify করতে পারে
- `passwd` setuid হওয়ায় temporarily root হিসেবে চলে

### Setgid (Set Group ID):

```bash
# Setgid bit set করো (directory তে)
chmod g+s /shared

ls -ld /shared
# drwxrwsr-x 2 root developers ...
#       ↑
#     's' = setgid

# এই directory তে file create করলে
# file এর group = directory এর group (developers)
```

### Sticky Bit:

```bash
# Sticky bit set করো
chmod +t /tmp

ls -ld /tmp
# drwxrwxrwt 10 root root ...
#         ↑
#       't' = sticky bit

# যেকোনো user file create করতে পারবে
# কিন্তু শুধু owner delete করতে পারবে
```

**Example:** `/tmp` directory:
- সবাই file create করতে পারে
- কিন্তু নিজের file শুধু নিজেই delete করতে পারবে

---

## ৮. Docker এ Users এবং Permissions

### Container Default User:

```dockerfile
FROM ubuntu:22.04

# Default: root user
RUN whoami  # Output: root
```

```bash
docker run ubuntu whoami
# Output: root
```

### Security Risk: Running as Root

```dockerfile
# ❌ Bad: root হিসেবে চলছে
FROM ubuntu:22.04
COPY app.py /app/
CMD ["python3", "/app/app.py"]
```

**সমস্যা:**
- Container compromise হলে root access পাবে
- Host system risk
- Security best practice violation

### ✅ Good: Non-Root User

```dockerfile
FROM ubuntu:22.04

# User তৈরি করো
RUN useradd -m -u 1001 -s /bin/bash appuser

# Ownership set করো
COPY --chown=appuser:appuser app.py /app/app.py

# Switch to non-root user
USER appuser

# এখন এই user হিসেবে চলবে
CMD ["python3", "/app/app.py"]
```

```bash
# Verify
docker run myimage whoami
# Output: appuser
```

---

## ৯. Kubernetes এ Users এবং Permissions

### SecurityContext:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  securityContext:
    runAsUser: 1001        # UID
    runAsGroup: 1001       # GID
    fsGroup: 1001          # Volume files এর group
    runAsNonRoot: true     # Root নিষিদ্ধ

  containers:
  - name: myapp
    image: myapp:v1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

### Volume Permissions:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  securityContext:
    fsGroup: 2000    # Volume files এ 2000 group set হবে

  containers:
  - name: myapp
    image: myapp
    volumeMounts:
    - name: data
      mountPath: /data

  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: myclaim
```

```bash
# Check inside container
kubectl exec mypod -- ls -ld /data
# drwxrwsr-x 2 root 2000 ...
#                   ↑
#              fsGroup applied!
```

---

## ১০. Practical Exercise

### Task 1: User এবং Group Management

```bash
# 1. নতুন group
sudo groupadd myteam

# 2. নতুন user
sudo useradd -m -G myteam alice
sudo passwd alice

# 3. Verify
id alice
groups alice

# 4. Shared directory তৈরি করো
sudo mkdir /shared
sudo chown root:myteam /shared
sudo chmod 770 /shared
# rwxrwx--- : শুধু myteam members access করতে পারবে

# 5. Test
sudo su - alice
cd /shared
touch myfile.txt
ls -l
# Owner: alice, Group: myteam

# 6. অন্য user try করুক (fail করবে)
exit
cd /shared   # Permission denied (যদি তুমি myteam এ না থাকো)
```

### Task 2: File Permissions Practice

```bash
# 1. Test files তৈরি করো
touch public.txt private.txt executable.sh

# 2. Public file (সবাই পড়তে পারবে)
chmod 644 public.txt
ls -l public.txt
# -rw-r--r--

# 3. Private file (শুধু owner)
chmod 600 private.txt
ls -l private.txt
# -rw-------

# 4. Executable (সবাই চালাতে পারবে)
chmod 755 executable.sh
ls -l executable.sh
# -rwxr-xr-x

# 5. Test করো
cat public.txt    # ✅ Works
cat private.txt   # ✅ Works (you're owner)

# অন্য user হিসেবে:
sudo su - alice
cat /path/to/public.txt   # ✅ Works
cat /path/to/private.txt  # ❌ Permission denied
```

### Task 3: Docker Non-Root

```bash
# 1. Bad Dockerfile
cat > Dockerfile.bad << 'EOF'
FROM ubuntu:22.04
COPY app.py /app/
CMD ["python3", "/app/app.py"]
EOF

docker build -t myapp:bad -f Dockerfile.bad .
docker run myapp:bad whoami
# Output: root  ❌

# 2. Good Dockerfile
cat > Dockerfile.good << 'EOF'
FROM ubuntu:22.04

# Create user
RUN useradd -m -u 1001 appuser

# Copy with ownership
COPY --chown=appuser:appuser app.py /app/

# Switch user
USER appuser

CMD ["python3", "/app/app.py"]
EOF

docker build -t myapp:good -f Dockerfile.good .
docker run myapp:good whoami
# Output: appuser  ✅
```

---

## ১১. Common Issues

### Issue 1: Permission Denied

```bash
# Error
bash: ./script.sh: Permission denied

# Check
ls -l script.sh
# -rw-r--r--  ← No execute permission!

# Fix
chmod +x script.sh
ls -l script.sh
# -rwxr-xr-x  ✅

# এখন run করো
./script.sh
```

### Issue 2: File Access Denied

```bash
# Error
cat /var/log/auth.log
# Permission denied

# Check
ls -l /var/log/auth.log
# -rw-r----- 1 syslog adm ...
#   ↑        ↑
# শুধু syslog user এবং adm group

# Fix: Root হিসেবে read করো
sudo cat /var/log/auth.log
```

### Issue 3: Docker Volume Permissions

```bash
# Host directory mount করো
docker run -v /host/data:/container/data myimage

# Container এ:
# touch /container/data/file.txt
# Permission denied!

# কেন? Host directory root owned, container non-root user

# Fix 1: Host এ permissions change করো
chmod 777 /host/data

# Fix 2: fsGroup use করো (Kubernetes)
# fsGroup: 1001
```

---

## ১২. User/Group Files

### /etc/passwd:

```bash
cat /etc/passwd

# Format:
username:x:UID:GID:comment:home:shell

# Example:
root:x:0:0:root:/root:/bin/bash
apple:x:1000:1000:Apple User:/home/apple:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

### /etc/group:

```bash
cat /etc/group

# Format:
groupname:x:GID:members

# Example:
root:x:0:
sudo:x:27:apple
developers:x:1001:alice,bob
```

### /etc/shadow:

```bash
sudo cat /etc/shadow

# Format:
username:encrypted_password:last_change:...

# Example:
root:$6$xyz...:18900:0:99999:7:::
apple:$6$abc...:18901:0:99999:7:::

# Note: শুধু root পড়তে পারে
```

---

## ১৩. Commands Summary

### User Management:

```bash
useradd username          # User তৈরি
userdel username          # User delete
passwd username           # Password set
usermod -aG group user    # Group এ add
whoami                    # Current user
id username               # User info
su - username             # Switch user
```

### Group Management:

```bash
groupadd groupname        # Group তৈরি
groupdel groupname        # Group delete
groups username           # User এর groups
```

### Permissions:

```bash
chmod 755 file            # Numeric
chmod u+x file            # Symbolic (user +execute)
chmod g-w file            # Symbolic (group -write)
chmod o+r file            # Symbolic (others +read)

chown user file           # Owner change
chgrp group file          # Group change
chown user:group file     # Both change
chown -R user:group dir   # Recursive
```

---

## ১৪. Interview Questions

### Q1: 755 মানে কী?

**Answer:**
```
755 = rwxr-xr-x

7 (rwx) = Owner: read + write + execute
5 (r-x) = Group: read + execute
5 (r-x) = Others: read + execute

Common for: directories, executables
```

### Q2: Docker container default কোন user হিসেবে চলে?

**Answer:**
```
Default: root (UID 0)

Security best practice: non-root user
```

```dockerfile
USER 1001
```

### Q3: Kubernetes Pod এ runAsNonRoot কী করে?

**Answer:**
```yaml
securityContext:
  runAsNonRoot: true
```

যদি container root হিসেবে চালাতে চায়, Pod start হবে না। Security enforcement।

---

## ১৫. Summary

✅ **Users:**
- Root (UID 0) - সব power
- Normal users (UID 1000+)
- System users (UID 1-999)

✅ **Permissions:**
```
rwxrwxrwx
│││ │││ │││
│││ │││ └─ Others
│││ └── Group
└── Owner

r = read (4)
w = write (2)
x = execute (1)
```

✅ **Common Codes:**
```
644 = rw-r--r--  (files)
755 = rwxr-xr-x  (dirs/executables)
600 = rw-------  (private)
```

✅ **Docker/Kubernetes:**
```dockerfile
USER appuser     # Non-root
```

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1001
```

✅ **Commands:**
```bash
chmod 755 file
chown user:group file
id, whoami, groups
```

**পরবর্তী পাঠ:** Logs এবং Monitoring! 🚀
