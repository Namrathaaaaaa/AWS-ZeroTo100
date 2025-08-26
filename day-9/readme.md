# AWS Load Balancers & OSI Model

## 🎯 What is Load Balancing?

**Load Balancing** distributes incoming traffic across multiple servers to ensure no single server gets overwhelmed, improving availability and performance.

**Benefits:** High availability • Better performance • Auto-scaling • Health monitoring

---

## 🌐 OSI 7-Layer Model

Understanding where load balancers operate in the network stack:

| Layer | Name             | Function                   | Examples         |
| ----- | ---------------- | -------------------------- | ---------------- |
| **7** | **Application**  | User interfaces, apps      | HTTP, HTTPS, FTP |
| **6** | **Presentation** | Data encryption/formatting | SSL/TLS, JPEG    |
| **5** | **Session**      | Manages connections        | NetBIOS, SQL     |
| **4** | **Transport**    | End-to-end delivery        | **TCP, UDP**     |
| **3** | **Network**      | Routing between networks   | **IP, ICMP**     |
| **2** | **Data Link**    | Node-to-node delivery      | Ethernet, WiFi   |
| **1** | **Physical**     | Physical transmission      | Cables, signals  |

**🔑 Key for Load Balancers:**

- **Layer 4 (Transport)**: Routes based on IP + Port
- **Layer 7 (Application)**: Routes based on content (HTTP headers, URLs)

---

## ⚖️ AWS Load Balancer Types

### 1. Application Load Balancer (ALB) - Layer 7

**Best for:** Web applications, HTTP/HTTPS traffic, microservices

**Features:**

- Content-based routing (URLs, headers)
- Host-based routing
- WebSocket support
- SSL termination
- Advanced request routing

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345 subnet-67890 \
  --security-groups sg-12345
```

**Use Cases:**

- ✅ Web applications with multiple services
- ✅ Microservices architecture
- ✅ Content-based routing needs
- ✅ API Gateway functionality

### 2. Network Load Balancer (NLB) - Layer 4

**Best for:** High-performance, low-latency, TCP/UDP traffic

**Features:**

- Ultra-high performance (millions of requests/sec)
- Static IP addresses
- Preserves source IP
- TCP/UDP load balancing
- Extreme low latency

```bash
# Create NLB
aws elbv2 create-load-balancer \
  --name my-nlb \
  --scheme internal \
  --type network \
  --subnets subnet-12345 subnet-67890
```

**Use Cases:**

- ✅ Gaming applications
- ✅ IoT applications
- ✅ High-frequency trading
- ✅ Static IP requirements
- ✅ Non-HTTP protocols

### 3. Gateway Load Balancer (GWLB) - Layer 3

**Best for:** Third-party network appliances, security tools

**Features:**

- Transparent network gateway
- Traffic inspection
- Firewall integration
- Intrusion detection
- Load balances network appliances

```bash
# Create GWLB
aws elbv2 create-load-balancer \
  --name my-gwlb \
  --type gateway \
  --subnets subnet-12345 subnet-67890
```

**Use Cases:**

- ✅ Network security appliances
- ✅ Firewalls and IDS/IPS
- ✅ Traffic inspection tools
- ✅ Network monitoring
- ✅ DDoS protection

---

## 🏗️ Real-World Architecture Examples

### E-commerce Website (ALB)

```
Users → ALB → Target Groups:
                ├── /api/* → API Servers
                ├── /images/* → Image Servers
                └── /* → Web Servers
```

### Gaming Platform (NLB)

```
Game Clients → NLB → Game Servers
                  ├── TCP:3000 → Match Server
                  ├── UDP:3001 → Voice Chat
                  └── TCP:3002 → Leaderboard
```

### Enterprise Security (GWLB)

```
Internet → GWLB → Security Appliances → Internal Network
                    ├── Firewall 1
                    ├── Firewall 2
                    └── IDS/IPS
```

---

## 📊 Load Balancer Comparison

| Feature             | ALB (Layer 7) | NLB (Layer 4) | GWLB (Layer 3) |
| ------------------- | ------------- | ------------- | -------------- |
| **Performance**     | High          | Ultra-high    | High           |
| **Latency**         | ~100ms        | ~1ms          | Low            |
| **Protocols**       | HTTP/HTTPS    | TCP/UDP/TLS   | All IP traffic |
| **Routing**         | Content-based | IP/Port-based | Transparent    |
| **SSL Termination** | ✅ Yes        | ✅ Yes        | ❌ No          |
| **Static IP**       | ❌ No         | ✅ Yes        | ✅ Yes         |
| **WebSockets**      | ✅ Yes        | ✅ Yes        | ✅ Yes         |
| **Cost**            | Moderate      | Low           | Moderate       |

---

## 🛠️ Configuration Examples

### ALB with Path-Based Routing

```yaml
# ALB Target Groups
/api/users → User Service (Port 3001)
/api/orders → Order Service (Port 3002)
/static/* → CDN/S3
/* → Frontend App (Port 80)
```

### NLB with Health Checks

```bash
# Create target group for NLB
aws elbv2 create-target-group \
  --name game-servers \
  --protocol TCP \
  --port 3000 \
  --vpc-id vpc-12345 \
  --health-check-protocol TCP \
  --health-check-port 3000
```

### GWLB with Security Integration

```yaml
# Security inspection flow
Internet Traffic → GWLB Endpoint → Security Appliance →
Application Subnets → Response through same path
```

---

## 🔐 Security & Best Practices

### SSL/TLS Configuration

```bash
# Add HTTPS listener to ALB
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:...
```

### Security Groups

```json
{
  "GroupName": "ALB-SecurityGroup",
  "Description": "Security group for ALB",
  "SecurityGroupRules": [
    {
      "IpProtocol": "tcp",
      "FromPort": 80,
      "ToPort": 80,
      "CidrIp": "0.0.0.0/0"
    },
    {
      "IpProtocol": "tcp",
      "FromPort": 443,
      "ToPort": 443,
      "CidrIp": "0.0.0.0/0"
    }
  ]
}
```

---

## 📈 Monitoring & Troubleshooting

### Key Metrics

```bash
# CloudWatch metrics to monitor
RequestCount          # Total requests
TargetResponseTime     # Backend response time
HTTPCode_Target_2XX    # Successful responses
UnHealthyHostCount     # Failed health checks
ActiveConnectionCount  # Current connections (NLB)
```

### Common Issues

| Issue                   | Load Balancer | Solution                                      |
| ----------------------- | ------------- | --------------------------------------------- |
| **High latency**        | ALB           | Check target response time, optimize backends |
| **Connection timeouts** | NLB           | Verify security groups and NACLs              |
| **503 errors**          | ALB           | Check target health, scaling policies         |
| **Unhealthy targets**   | All           | Review health check configuration             |

---

## 💰 Cost Optimization

### Pricing Overview

- **ALB**: $0.0225/hour + $0.008/LCU-hour
- **NLB**: $0.0225/hour + $0.006/NLCU-hour
- **GWLB**: $0.0125/hour + $0.004/GLCU-hour

### Cost Tips

```bash
# Use target group deregistration delay
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --attributes Key=deregistration_delay.timeout_seconds,Value=30

# Enable access logs only when needed
# Use appropriate health check intervals
```

---

## 🎯 When to Use Each Type

### Choose ALB when:

- ✅ HTTP/HTTPS applications
- ✅ Microservices with different endpoints
- ✅ Need content-based routing
- ✅ WebSocket support required
- ✅ SSL termination needed

### Choose NLB when:

- ✅ Non-HTTP protocols (TCP/UDP)
- ✅ Ultra-low latency required
- ✅ Static IP addresses needed
- ✅ High-performance applications
- ✅ Gaming or IoT applications

### Choose GWLB when:

- ✅ Network security appliances
- ✅ Traffic inspection required
- ✅ Third-party security tools
- ✅ Centralized security policies
- ✅ Compliance requirements

---

## 🚀 Quick Decision Tree

```
Need security appliance integration? → GWLB
    ↓ No
Need HTTP/HTTPS routing? → ALB
    ↓ No
Need ultra-high performance? → NLB
    ↓ No
Default choice → ALB
```

---

**🎯 AWS Load Balancers: Choose the right layer for your application needs - Layer 7 for intelligence, Layer 4 for speed, Layer 3 for security!**
