# Networking Basics - নেটওয়ার্কিং মৌলিক ধারণা

## ১. কেন দরকার? (Why?)

ধরো তুমি কাউকে চিঠি পাঠাতে চাও। দরকার কী?
- **ঠিকানা** (address)
- **পোস্ট অফিস** (router)
- **ডাকপিয়ন** (packet delivery)

Computer networking-ও একই! একটা computer অন্যটার সাথে communicate করতে এই concepts দরকার।

### Kubernetes এ কেন দরকার?

- Pod কীভাবে একে অপরের সাথে talk করে?
- Service কী এবং কেন?
- Ingress কীভাবে কাজ করে?
- Port mapping, DNS, load balancing

Networking না বুঝলে Kubernetes debugging impossible!

---

## ২. IP Address - আইপি ঠিকানা

### IP Address কী?

**IP Address = Network এ একটা device এর unique identifier।**

চিঠির ঠিকানার মতো:
```
বাড়ি নং ১০, রোড ৫, ঢাকা
    ↓
192.168.1.10
```

### IP Address Format:

**IPv4:**
```
192.168.1.10
 │   │  │ │
 └───┴──┴─┴── চার অংশ, প্রতিটা 0-255
```

**IPv6:** (নতুন, বড়)
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

### IP Types:

#### Private IP (ঘরের ভিতর):

```
10.0.0.0      - 10.255.255.255
172.16.0.0    - 172.31.255.255
192.168.0.0   - 192.168.255.255
```

**কখন ব্যবহার:**
- Home network
- Office network
- Docker/Kubernetes cluster (internally)

#### Public IP (বাইরের দুনিয়া):

```
8.8.8.8           - Google DNS
142.250.190.78    - google.com
```

**কখন ব্যবহার:**
- Internet access
- Public websites
- Cloud servers (external)

---

## ৩. Localhost - নিজের Computer

### Localhost কী?

```bash
localhost = 127.0.0.1 = "আমার নিজের computer"
```

যখন application নিজের computer এ run হচ্ছে:

```python
# Server চালাও
app.run(host="127.0.0.1", port=8080)

# Access করো
http://localhost:8080
http://127.0.0.1:8080    # Same!
```

### Loopback Interface:

```
Application A          Application B
(localhost:8080)       (localhost:3000)
       ↓                     ↓
   127.0.0.1         127.0.0.1
       └─────────────────┘
         Same computer!
```

### /etc/hosts:

```bash
cat /etc/hosts

# Output:
127.0.0.1   localhost
127.0.1.1   mycomputer
::1         localhost  # IPv6
```

---

## ৪. Port - পোর্ট

### Port কী?

একটা IP address এ অনেক application চলতে পারে। Port দিয়ে distinguish করো:

```
Building (IP Address)
├── Apartment 80  (Port 80)  → Web server
├── Apartment 22  (Port 22)  → SSH
├── Apartment 3306 (Port 3306) → MySQL
└── Apartment 8080 (Port 8080) → App server
```

### Format:

```
IP:Port
192.168.1.10:80
192.168.1.10:8080
localhost:3000
```

### Port Ranges:

| Range | Type | Example |
|-------|------|---------|
| 0-1023 | Well-known | 80 (HTTP), 443 (HTTPS), 22 (SSH) |
| 1024-49151 | Registered | 3306 (MySQL), 5432 (PostgreSQL) |
| 49152-65535 | Dynamic | Random ports |

### Common Ports:

```
22    - SSH
80    - HTTP
443   - HTTPS
3000  - Node.js default
3306  - MySQL
5432  - PostgreSQL
6379  - Redis
8080  - Alternative HTTP
```

---

## ৫. TCP vs UDP

### TCP (Transmission Control Protocol):

**বিশ্বস্ত** - registered post এর মতো:

```
Client                  Server
  |                        |
  |---- SYN ----------->   |  1. Connection request
  |<--- SYN-ACK ---------|  2. Acknowledge
  |---- ACK ------------>|  3. Connection established
  |                        |
  |---- Data ------------>|  4. Send data
  |<--- ACK --------------|  5. Confirm received
  |                        |
  |---- FIN ------------>|  6. Close connection
  |<--- ACK --------------|  7. Confirm close
```

**বৈশিষ্ট্য:**
- ✅ Reliable (data হারায় না)
- ✅ Ordered (order maintain করে)
- ✅ Error checking
- ❌ Slower (overhead আছে)

**Use cases:** HTTP, SSH, Database connections

### UDP (User Datagram Protocol):

**দ্রুত কিন্তু unreliable** - SMS এর মতো:

```
Client                  Server
  |                        |
  |---- Data ------------>|  Send and forget!
  |---- Data ------------>|  
  |---- Data ------------>|  
```

**বৈশিষ্ট্য:**
- ✅ Fast
- ✅ Low overhead
- ❌ Unreliable (packet হারাতে পারে)
- ❌ No order guarantee

**Use cases:** DNS, Video streaming, Gaming

---

## ৬. DNS (Domain Name System)

### DNS কী?

**DNS = Phone book for internet**

মানুষ মনে রাখে: `google.com`  
Computer চায়: `142.250.190.78`

DNS translate করে: name → IP

### DNS Lookup:

```
You type: https://google.com
           ↓
1. Browser asks: "What is google.com IP?"
           ↓
2. DNS Server: "It's 142.250.190.78"
           ↓
3. Browser connects to 142.250.190.78:443
```

### Hands-On:

```bash
# Domain এর IP খুঁজো
nslookup google.com

# Output:
Server:    192.168.1.1
Address:   192.168.1.1#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.190.78

# Or using dig
dig google.com +short
# Output: 142.250.190.78

# Or using host
host google.com
```

### /etc/hosts (Local DNS):

```bash
# Edit
sudo nano /etc/hosts

# Add:
127.0.0.1   myapp.local
192.168.1.100   database.local

# এখন use করো:
ping myapp.local
curl http://myapp.local:8080
```

---

## ৭. HTTP/HTTPS

### HTTP (HyperText Transfer Protocol):

**Web এর ভাষা:**

```
Client                           Server
  |                                 |
  |--- GET /index.html HTTP/1.1 -->|  Request
  |    Host: example.com            |
  |                                 |
  |<-- HTTP/1.1 200 OK ------------|  Response
  |    Content-Type: text/html     |
  |    <html>...</html>             |
```

### HTTP Request:

```http
GET /api/users HTTP/1.1
Host: api.example.com
User-Agent: curl/7.68.0
Accept: application/json
Authorization: Bearer token123
```

**Components:**
- **Method:** GET, POST, PUT, DELETE
- **Path:** /api/users
- **Headers:** Key-value pairs
- **Body:** Data (for POST/PUT)

### HTTP Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 1234

{"users": [...]}
```

**Components:**
- **Status Code:** 200, 404, 500
- **Headers:** Metadata
- **Body:** Actual data

### HTTP Methods:

| Method | কাজ | Example |
|--------|-----|---------|
| GET | Read data | Get user list |
| POST | Create | Create new user |
| PUT | Update (full) | Update user |
| PATCH | Update (partial) | Update user name |
| DELETE | Delete | Delete user |

### HTTPS (Secure):

```
HTTP  → Plain text → ❌ Insecure
HTTPS → Encrypted  → ✅ Secure (TLS/SSL)
```

```
http://example.com     - Port 80
https://example.com    - Port 443
```

---

## ৮. Request/Response Cycle

### Full Flow:

```
Browser                DNS              Server
   |                    |                  |
   |-- google.com --->  |                  |  1. DNS lookup
   |<- 142.250.x.x --|  |                  |
   |                    |                  |
   |---------- TCP Connection ----------->|  2. Connect
   |                                       |
   |-------- GET / HTTP/1.1 ------------->|  3. HTTP request
   |                                       |
   |<------- HTML response --------------|  4. HTTP response
   |                                       |
   |-------- Close connection ----------->|  5. Done
```

### Hands-On:

```bash
# HTTP request using curl
curl -v http://example.com

# Output shows:
* Trying 93.184.216.34:80...
* Connected
> GET / HTTP/1.1
> Host: example.com
>
< HTTP/1.1 200 OK
< Content-Type: text/html
<
<!doctype html>
<html>...</html>
```

---

## ৯. Docker/Kubernetes Networking

### Docker Network:

```bash
# Default bridge network
docker run -d -p 8080:80 nginx

# Explanation:
Host:8080  →  Container:80
   ↓              ↓
Your computer   Nginx inside container
```

**যখন access করো:**
```
http://localhost:8080
        ↓
    Docker maps to
        ↓
    Container:80
        ↓
    Nginx responds
```

### Container-to-Container:

```bash
# একই network এ
docker network create mynetwork
docker run -d --name db --network mynetwork postgres
docker run -d --name app --network mynetwork myapp

# app container থেকে:
# postgres://db:5432  ← hostname = container name!
```

### Kubernetes Networking:

#### Pod IP:

```bash
kubectl get pods -o wide

# Output:
NAME    READY   STATUS    IP
pod-1   1/1     Running   10.244.1.5
pod-2   1/1     Running   10.244.1.6
```

প্রতিটা Pod এর নিজস্ব IP আছে।

#### Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80           # Service port
    targetPort: 8080   # Pod port
```

```
User → Service IP:80 → Pod:8080
```

---

## ১০. Practical Exercise

### Task 1: IP এবং Networking Commands

```bash
# 1. নিজের IP দেখো
ip addr show
# Or
ifconfig

# 2. Default gateway
ip route show
# Or
route -n

# 3. DNS servers
cat /etc/resolv.conf

# 4. Ping test
ping -c 3 google.com

# 5. Traceroute (path দেখো)
traceroute google.com
```

### Task 2: Port Testing

```bash
# 1. কোন ports open?
sudo netstat -tulpn

# Or
sudo ss -tulpn

# 2. Specific port test
nc -zv localhost 80
telnet localhost 80

# 3. Python web server চালাও
python3 -m http.server 8000
# Access: http://localhost:8000

# 4. Port already in use?
lsof -i :8000
```

### Task 3: HTTP Request/Response

```bash
# 1. Simple GET request
curl http://example.com

# 2. Verbose (দেখো কী হচ্ছে)
curl -v http://example.com

# 3. POST request
curl -X POST http://example.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# 4. Follow redirects
curl -L http://google.com

# 5. Save to file
curl -o output.html http://example.com
```

### Task 4: Docker Networking

```bash
# 1. Container run করো
docker run -d -p 8080:80 --name web nginx

# 2. Access করো
curl http://localhost:8080

# 3. Container এর IP দেখো
docker inspect web | grep IPAddress

# 4. Host থেকে container IP তে access
curl http://<container-ip>:80

# 5. Network list
docker network ls

# 6. Custom network
docker network create mynet
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp

# app থেকে db access:
docker exec app ping db    # ✅ Works!
```

---

## ১১. Common Commands Summary

### IP/Network Info:

```bash
ip addr show           # IP addresses
ip route show          # Routing table
netstat -i             # Network interfaces
ifconfig               # Network config (old)
```

### DNS:

```bash
nslookup domain.com    # DNS lookup
dig domain.com         # Detailed DNS
host domain.com        # Simple DNS
cat /etc/resolv.conf   # DNS servers
```

### Connectivity:

```bash
ping host              # Test reachability
traceroute host        # Show route
telnet host port       # Test port
nc -zv host port       # Port scan
```

### Ports:

```bash
netstat -tulpn         # Open ports
ss -tulpn              # Open ports (modern)
lsof -i :8080          # What's using port 8080
```

---

## ১২. Interview Questions

### Q1: localhost vs 0.0.0.0?

**Answer:**

```python
# localhost (127.0.0.1)
app.run(host="127.0.0.1")
# শুধু local machine থেকে access

# 0.0.0.0 (all interfaces)
app.run(host="0.0.0.0")
# Network এর যেকোনো IP থেকে access
```

### Q2: Docker -p vs -P?

**Answer:**

```bash
# -p (manual mapping)
docker run -p 8080:80 nginx
# Host:8080 → Container:80

# -P (auto mapping)
docker run -P nginx
# Docker randomly assigns host port
```

### Q3: TCP vs UDP কখন কোনটা?

**Answer:**

**TCP:** যখন data loss হওয়া চলবে না
- HTTP, HTTPS
- Database connections
- File transfers

**UDP:** যখন speed দরকার, কিছু loss ঠিক আছে
- Video streaming
- Gaming
- DNS queries

### Q4: Kubernetes Service কেন দরকার?

**Answer:**

```
Without Service:
Pod IP: 10.244.1.5  → Pod restart হলে IP change!

With Service:
Service IP: 10.96.0.10  → স্থায়ী IP
  ↓
Load balance to Pods (যেকোনো IP)
```

---

## ১৩. Summary

✅ **IP Address:**
- Device এর unique identifier
- Private: 192.168.x.x, 10.x.x.x (local)
- Public: Internet access

✅ **Port:**
- Application identifier
- IP:Port format
- 1-65535 range

✅ **TCP vs UDP:**
- TCP: Reliable, ordered, slower
- UDP: Fast, unreliable

✅ **DNS:**
- Domain → IP translation
- google.com → 142.250.190.78

✅ **HTTP/HTTPS:**
- Request/Response protocol
- Methods: GET, POST, PUT, DELETE
- HTTPS: Encrypted

✅ **Docker:**
```bash
docker run -p 8080:80    # Port mapping
--network mynet          # Custom network
```

✅ **Kubernetes:**
```yaml
Service → Load balances → Pods
```

**পরবর্তী পাঠ:** আরো detailed networking এবং Docker concepts! 🚀
