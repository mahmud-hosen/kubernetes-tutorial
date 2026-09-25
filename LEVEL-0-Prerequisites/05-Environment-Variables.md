# Environment Variables - পরিবেশ ভেরিয়েবল

## ১. কেন দরকার? (Why?)

ধরো তুমি একটা mobile app বানাচ্ছো। এই app:
- **Development** এ test server use করবে
- **Production** এ real server use করবে

Code একই, কিন্তু server address আলাদা! হার্ডকোড করবে নাকি?

```python
# ❌ Bad - Hardcoded
API_URL = "https://api.production.com"

# ✅ Good - Environment Variable
import os
API_URL = os.getenv("API_URL")
```

**Environment Variables = Configuration যা code এর বাইরে থেকে দেওয়া হয়।**

### Benefits:

1. **Same code, different environments**
2. **Secrets secure রাখা** (password, API key)
3. **Configuration flexibility**
4. **No code change** for different settings

### Kubernetes এ কেন দরকার?

- Pod এ database URL, API keys pass করতে
- ConfigMap/Secret data inject করতে
- Container behavior customize করতে
- Multi-environment (dev/staging/prod) manage করতে

---

## ২. কী? (What?)

**Environment Variable = Key-Value pair যা process এর environment এ থাকে।**

```bash
KEY=VALUE
```

### Examples:

```bash
HOME=/home/apple
USER=apple
PATH=/usr/bin:/bin
DATABASE_URL=postgresql://localhost/mydb
API_KEY=secret123
```

### Scope:

```
System-wide
    ↓
User-level (/etc/profile, ~/.bashrc)
    ↓
Shell session
    ↓
Process
    ↓
Child process (inherited)
```

---

## ৩. কীভাবে কাজ করে? (How?)

### Environment Inheritance:

```
Parent Process
  ENV: HOME=/home/apple
       PATH=/usr/bin
       USER=apple
       ↓
  fork() - Creates child
       ↓
Child Process (inherits)
  ENV: HOME=/home/apple    ← Same!
       PATH=/usr/bin       ← Same!
       USER=apple          ← Same!
       MY_VAR=hello        ← Can add new
```

### Memory Layout:

```
Process Memory
├── Code
├── Data
├── Heap
├── Stack
└── Environment          ← ENV variables এখানে
    ├── HOME=/home/apple
    ├── PATH=/usr/bin
    └── USER=apple
```

---

## ৪. Hands-On Example

### Example 1: View Environment Variables

```bash
# সব environment variables
env

# Or
printenv

# Output:
HOME=/home/apple
USER=apple
PATH=/usr/local/bin:/usr/bin:/bin
SHELL=/bin/bash
PWD=/home/apple
LANG=en_US.UTF-8
...

# Specific variable
echo $HOME
echo $USER
echo $PATH

# Or
printenv HOME
printenv USER
```

### Example 2: Set Environment Variable

```bash
# Current shell only
export MY_VAR="Hello World"
echo $MY_VAR
# Output: Hello World

# Verify
env | grep MY_VAR
# Output: MY_VAR=Hello World

# নতুন terminal খুললে?
# MY_VAR নেই! কারণ শুধু current session এর জন্য
```

### Example 3: Temporary Variable (One Command)

```bash
# শুধু এই command এর জন্য
MY_VAR="test" python3 -c "import os; print(os.getenv('MY_VAR'))"
# Output: test

# পরে check করো
echo $MY_VAR
# Output: (empty - কারণ set করিনি)
```

### Example 4: Persistent Variable

```bash
# ~/.bashrc এ add করো
echo 'export MY_VAR="permanent"' >> ~/.bashrc

# Reload
source ~/.bashrc

# Check
echo $MY_VAR
# Output: permanent

# নতুন terminal খুললেও থাকবে!
```

---

## ৫. Common Environment Variables

### System Variables:

| Variable | মানে | Example |
|----------|------|---------|
| `HOME` | User home directory | `/home/apple` |
| `USER` | Current username | `apple` |
| `PWD` | Current directory | `/home/apple/projects` |
| `OLDPWD` | Previous directory | `/home/apple` |
| `SHELL` | Current shell | `/bin/bash` |
| `TERM` | Terminal type | `xterm-256color` |
| `LANG` | Language/locale | `en_US.UTF-8` |
| `PATH` | Executable search path | `/usr/bin:/bin` |

### Special Variables:

```bash
# Exit status of last command
echo $?
# 0 = success, non-zero = error

# Current shell PID
echo $$

# Last background process PID
sleep 100 &
echo $!

# Number of arguments
echo $#

# All arguments
echo $@
```

### Application Variables:

```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# API
API_KEY=secret123
API_URL=https://api.example.com

# Application
DEBUG=true
LOG_LEVEL=info
PORT=8080
```

---

## ৬. PATH Variable (বিশেষ গুরুত্বপূর্ণ)

### PATH কী?

```bash
echo $PATH
# Output:
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

যখন তুমি একটা command চালাও:

```bash
ls
```

Shell এই directories তে খোঁজে (order এ):
1. `/usr/local/bin/ls` - নেই
2. `/usr/bin/ls` - **পেয়েছি!** ✅
3. Run করো: `/usr/bin/ls`

### PATH Modify করো:

```bash
# Current PATH দেখো
echo $PATH

# নতুন directory add করো (সামনে)
export PATH="/my/custom/bin:$PATH"

# নতুন directory add করো (শেষে)
export PATH="$PATH:/another/bin"

# Verify
echo $PATH
```

### Why PATH Matters:

```bash
# Without PATH:
/usr/bin/python3 script.py    # Full path দিতে হবে

# With PATH:
python3 script.py             # শুধু command name!
```

---

## ৭. Docker এ Environment Variables

### Dockerfile:

```dockerfile
FROM ubuntu:22.04

# Build-time (ARG)
ARG BUILD_DATE=2024

# Runtime (ENV)
ENV APP_ENV=production
ENV PORT=8080
ENV DATABASE_URL=postgresql://db:5432/mydb

COPY app.py /app/app.py

CMD python3 /app/app.py
```

### Docker Run:

```bash
# Single variable
docker run -e MY_VAR=value myimage

# Multiple variables
docker run -e VAR1=val1 -e VAR2=val2 myimage

# From file
cat > env.list << EOF
DATABASE_URL=postgresql://localhost/db
API_KEY=secret123
DEBUG=true
EOF

docker run --env-file env.list myimage

# Check inside container
docker exec mycontainer env
```

### Example App:

```python
# app.py
import os

database_url = os.getenv("DATABASE_URL", "default_value")
debug = os.getenv("DEBUG", "false") == "true"
port = int(os.getenv("PORT", "8080"))

print(f"Database: {database_url}")
print(f"Debug: {debug}")
print(f"Port: {port}")
```

```bash
# Run with custom env
docker run -e DATABASE_URL=postgres://prod/db \
           -e DEBUG=false \
           -e PORT=9000 \
           myapp

# Output:
# Database: postgres://prod/db
# Debug: False
# Port: 9000
```

---

## ৮. Kubernetes এ Environment Variables

### Direct Value:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: myapp
    image: myapp
    env:
    - name: DATABASE_URL
      value: "postgresql://db:5432/mydb"
    - name: DEBUG
      value: "true"
    - name: LOG_LEVEL
      value: "info"
```

```bash
# Check
kubectl exec mypod -- env | grep DATABASE_URL
```

### From ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: myapp
    image: myapp
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log_level
```

### From Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  api_key: c2VjcmV0MTIz    # base64 encoded
  db_password: cGFzc3dvcmQ=
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: myapp
    image: myapp
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: api_key
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: db_password
```

### All ConfigMap as ENV:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: myapp
    image: myapp
    envFrom:
    - configMapRef:
        name: app-config    # সব keys environment variable হবে
```

---

## ৯. Practical Exercise

### Task 1: Basic ENV Usage

```bash
# 1. Set করো
export MY_NAME="Apple"
export MY_AGE=25

# 2. Use করো
echo "Name: $MY_NAME, Age: $MY_AGE"

# 3. Script লিখো
cat > greet.sh << 'EOF'
#!/bin/bash
echo "Hello, $MY_NAME!"
echo "You are $MY_AGE years old"
EOF

chmod +x greet.sh
./greet.sh

# 4. Different value দিয়ে run করো
MY_NAME="John" ./greet.sh
```

### Task 2: Python ENV

```bash
# 1. Python script
cat > app.py << 'EOF'
import os

# With default value
name = os.getenv("NAME", "Guest")
age = os.getenv("AGE", "Unknown")

print(f"Hello {name}, age {age}")

# Required variable (error if not set)
api_key = os.environ["API_KEY"]  # KeyError if missing
print(f"API Key: {api_key}")
EOF

# 2. Without API_KEY (will fail)
python3 app.py
# KeyError: 'API_KEY'

# 3. With all variables
NAME="Apple" AGE=25 API_KEY=secret123 python3 app.py
# Output:
# Hello Apple, age 25
# API Key: secret123
```

### Task 3: Docker ENV

```bash
# 1. Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.9-slim

ENV DEFAULT_VAR="from-dockerfile"

COPY app.py /app/app.py
WORKDIR /app

CMD ["python3", "app.py"]
EOF

# 2. app.py
cat > app.py << 'EOF'
import os
print("DEFAULT_VAR:", os.getenv("DEFAULT_VAR"))
print("RUNTIME_VAR:", os.getenv("RUNTIME_VAR"))
EOF

# 3. Build
docker build -t envtest .

# 4. Run without override
docker run envtest
# DEFAULT_VAR: from-dockerfile
# RUNTIME_VAR: None

# 5. Run with override
docker run -e DEFAULT_VAR=overridden -e RUNTIME_VAR=runtime envtest
# DEFAULT_VAR: overridden
# RUNTIME_VAR: runtime
```

### Task 4: Kubernetes ENV

```bash
# 1. ConfigMap তৈরি করো
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=postgres://db/mydb \
  --from-literal=LOG_LEVEL=debug

# 2. Pod তৈরি করো
cat > pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: envtest
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: CUSTOM_VAR
      value: "my-value"
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_URL
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
EOF

kubectl apply -f pod.yaml

# 3. Check করো
kubectl logs envtest | grep -E 'CUSTOM_VAR|DATABASE_URL|LOG_LEVEL'

# Output:
# CUSTOM_VAR=my-value
# DATABASE_URL=postgres://db/mydb
# LOG_LEVEL=debug
```

---

## ১০. Common Patterns

### 12-Factor App:

```python
# ✅ Good - Config from ENV
import os

class Config:
    DATABASE_URL = os.getenv("DATABASE_URL")
    SECRET_KEY = os.getenv("SECRET_KEY")
    DEBUG = os.getenv("DEBUG", "False") == "True"
    PORT = int(os.getenv("PORT", "8000"))
    
    # Validation
    if not DATABASE_URL:
        raise ValueError("DATABASE_URL is required")
```

### Default Values:

```bash
# Bash
DATABASE_URL=${DATABASE_URL:-"postgresql://localhost/db"}
DEBUG=${DEBUG:-false}

# Python
import os
database_url = os.getenv("DATABASE_URL") or "default_url"
```

### Required vs Optional:

```python
import os

# Required (will raise error if not set)
api_key = os.environ["API_KEY"]

# Optional (will use default)
log_level = os.getenv("LOG_LEVEL", "info")
```

---

## ১১. Security Best Practices

### ❌ Never Hardcode Secrets:

```python
# ❌ Bad
API_KEY = "sk_live_abc123xyz"
DATABASE_PASSWORD = "mysecretpass"

# ✅ Good
API_KEY = os.getenv("API_KEY")
DATABASE_PASSWORD = os.getenv("DB_PASSWORD")
```

### ❌ Don't Log Secrets:

```python
# ❌ Bad
print(f"API Key: {os.getenv('API_KEY')}")

# ✅ Good
api_key = os.getenv("API_KEY")
if api_key:
    print("API Key: [REDACTED]")
```

### ✅ Use Secret Management:

```yaml
# Kubernetes Secret (better than ConfigMap for secrets)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  api-key: "secret123"    # Will be base64 encoded automatically
```

---

## ১২. Troubleshooting

### Issue 1: Variable Not Set

```bash
# Check if set
echo $MY_VAR
# Empty = not set

# Or
printenv MY_VAR
# Nothing = not set

# In script:
if [ -z "$MY_VAR" ]; then
    echo "MY_VAR is not set!"
fi
```

### Issue 2: Variable Not Inherited

```bash
# ❌ Without export
MY_VAR="hello"
bash -c 'echo $MY_VAR'
# Empty! Child bash doesn't inherit

# ✅ With export
export MY_VAR="hello"
bash -c 'echo $MY_VAR'
# hello - inherited!
```

### Issue 3: Docker ENV Not Working

```bash
# Check Dockerfile
ENV MY_VAR=value    # ✅ Runtime
ARG MY_VAR=value    # ❌ Build-time only

# Check if overridden
docker run -e MY_VAR=new_value myimage env | grep MY_VAR
```

---

## ১৩. Interview Questions

### Q1: ENV vs ARG in Dockerfile?

**Answer:**

```dockerfile
# ARG - Build time only
ARG BUILD_DATE=2024
RUN echo "Built on $BUILD_DATE"

# ENV - Runtime
ENV APP_ENV=production

# ARG ব্যবহার করে ENV set করো
ARG VERSION=1.0
ENV APP_VERSION=$VERSION
```

### Q2: Kubernetes এ ConfigMap vs Secret?

**Answer:**

| | ConfigMap | Secret |
|---|---|---|
| **Use for** | Non-sensitive config | Passwords, tokens |
| **Encoding** | Plain text | Base64 |
| **kubectl get** | Visible | Hidden by default |
| **Best for** | URLs, flags | API keys, certs |

### Q3: Environment variable inherit হয় কীভাবে?

**Answer:**

```bash
Parent Process
  export MY_VAR="hello"
     ↓
  fork() - Creates child
     ↓
Child Process
  MY_VAR="hello"    ← Inherited!

# কিন্তু:
Child Process modifies MY_VAR
     ↓
Parent এ change হয় না! (separate memory)
```

---

## ১৪. Summary

✅ **Environment Variables:**
- Key-Value pairs in process environment
- Configuration without code change
- Inherited from parent to child process

✅ **Common Uses:**
```bash
PATH=/usr/bin:/bin           # Where to find commands
HOME=/home/apple             # User home
DATABASE_URL=postgres://...  # App config
```

✅ **Docker:**
```dockerfile
ENV MY_VAR=value
```
```bash
docker run -e MY_VAR=value myimage
```

✅ **Kubernetes:**
```yaml
env:
- name: MY_VAR
  value: "direct-value"
- name: FROM_CONFIG
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: key-name
```

✅ **Commands:**
```bash
export MY_VAR="value"    # Set
echo $MY_VAR             # Read
env                      # List all
unset MY_VAR             # Remove
```

**পরবর্তী পাঠ:** Ports এবং Networking basics! 🚀
