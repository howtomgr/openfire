# Openfire Installation Guide

Openfire is a free and open-source XMPP Server. Cross-platform real-time collaboration server

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5222/9090 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5222/9090 (default openfire port)
- **Dependencies**:
  - java-11-openjdk
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install openfire
sudo dnf install -y openfire java-11-openjdk

# Enable and start service
sudo systemctl enable --now openfire

# Configure firewall
sudo firewall-cmd --permanent --add-service=openfire
sudo firewall-cmd --reload

# Verify installation
openfire --version || systemctl status openfire
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install openfire
sudo apt install -y openfire java-11-openjdk

# Enable and start service
sudo systemctl enable --now openfire

# Configure firewall
sudo ufw allow 5222/9090

# Verify installation
openfire --version || systemctl status openfire
```

### Arch Linux

```bash
# Install openfire
sudo pacman -S openfire

# Enable and start service
sudo systemctl enable --now openfire

# Verify installation
openfire --version || systemctl status openfire
```

### Alpine Linux

```bash
# Install openfire
apk add --no-cache openfire

# Enable and start service
rc-update add openfire default
rc-service openfire start

# Verify installation
openfire --version || rc-service openfire status
```

### openSUSE/SLES

```bash
# Install openfire
sudo zypper install -y openfire java-11-openjdk

# Enable and start service
sudo systemctl enable --now openfire

# Configure firewall
sudo firewall-cmd --permanent --add-service=openfire
sudo firewall-cmd --reload

# Verify installation
openfire --version || systemctl status openfire
```

### macOS

```bash
# Using Homebrew
brew install openfire

# Start service
brew services start openfire

# Verify installation
openfire --version
```

### FreeBSD

```bash
# Using pkg
pkg install openfire

# Enable in rc.conf
echo 'openfire_enable="YES"' >> /etc/rc.conf

# Start service
service openfire start

# Verify installation
openfire --version || service openfire status
```

### Windows

```powershell
# Using Chocolatey
choco install openfire

# Or using Scoop
scoop install openfire

# Verify installation
openfire --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/openfire

# Set up basic configuration
sudo tee /etc/openfire/openfire.conf << 'EOF'
# Openfire Configuration
-Xms512m -Xmx1024m
EOF

# Test configuration
sudo openfire -t || sudo openfire configtest

# Reload service
sudo systemctl reload openfire
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R openfire:openfire /etc/openfire
sudo chmod 750 /etc/openfire

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable openfire

# Start service
sudo systemctl start openfire

# Stop service
sudo systemctl stop openfire

# Restart service
sudo systemctl restart openfire

# Reload configuration
sudo systemctl reload openfire

# Check status
sudo systemctl status openfire

# View logs
sudo journalctl -u openfire -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add openfire default

# Start service
rc-service openfire start

# Stop service
rc-service openfire stop

# Restart service
rc-service openfire restart

# Check status
rc-service openfire status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'openfire_enable="YES"' >> /etc/rc.conf

# Start service
service openfire start

# Stop service
service openfire stop

# Restart service
service openfire restart

# Check status
service openfire status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start openfire
brew services stop openfire
brew services restart openfire

# Check status
brew services list | grep openfire
```

### Windows Service Manager

```powershell
# Start service
net start openfire

# Stop service
net stop openfire

# Using PowerShell
Start-Service openfire
Stop-Service openfire
Restart-Service openfire

# Check status
Get-Service openfire
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/openfire/openfire.conf << 'EOF'
-Xms512m -Xmx1024m
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart openfire
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream openfire_backend {
    server 127.0.0.1:5222/9090;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name openfire.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name openfire.example.com;

    ssl_certificate /etc/ssl/certs/openfire.example.com.crt;
    ssl_certificate_key /etc/ssl/private/openfire.example.com.key;

    location / {
        proxy_pass http://openfire_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName openfire.example.com
    Redirect permanent / https://openfire.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName openfire.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/openfire.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/openfire.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5222/9090/
    ProxyPassReverse / http://127.0.0.1:5222/9090/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5222/9090/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend openfire_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/openfire.pem
    redirect scheme https if !{ ssl_fc }
    default_backend openfire_backend

backend openfire_backend
    balance roundrobin
    option httpchk GET /health
    server openfire1 127.0.0.1:5222/9090 check
    server openfire2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R openfire:openfire /etc/openfire
sudo chmod 750 /etc/openfire

# Configure firewall
sudo firewall-cmd --permanent --add-service=openfire
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/openfire.conf << 'EOF'
[openfire]
enabled = true
port = 5222/9090
filter = openfire
logpath = /var/log/openfire/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/openfire.key \
    -out /etc/ssl/certs/openfire.crt

# Configure SSL in openfire
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE openfire_db;
CREATE USER openfire_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE openfire_db TO openfire_user;
EOF

# Configure openfire to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE openfire_db;
CREATE USER 'openfire_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON openfire_db.* TO 'openfire_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Openfire specific tuning
-Xms512m -Xmx1024m
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
openfire soft nofile 65535
openfire hard nofile 65535
openfire soft nproc 32768
openfire hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'openfire'
    static_configs:
      - targets: ['localhost:5222/9090']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet openfire; then
    echo "Openfire is running"
    exit 0
else
    echo "Openfire is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/openfire << 'EOF'
/var/log/openfire/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 openfire openfire
    postrotate
        systemctl reload openfire > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Openfire backup script
BACKUP_DIR="/backup/openfire"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop openfire

# Backup configuration
tar -czf "$BACKUP_DIR/openfire-config-$DATE.tar.gz" /etc/openfire

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/openfire-data-$DATE.tar.gz" /var/lib/openfire

# Start service
systemctl start openfire

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop openfire

# Restore configuration
sudo tar -xzf /backup/openfire/openfire-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/openfire/openfire-data-*.tar.gz -C /

# Set permissions
sudo chown -R openfire:openfire /etc/openfire
sudo chown -R openfire:openfire /var/lib/openfire

# Start service
sudo systemctl start openfire
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u openfire -n 100
sudo tail -f /var/log/openfire/*.log

# Check configuration
sudo openfire -t || sudo openfire configtest

# Check permissions
ls -la /etc/openfire
ls -la /var/lib/openfire
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5222/9090
sudo netstat -tlnp | grep 5222/9090

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5222/9090
nc -zv localhost 5222/9090
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep java)
htop -p $(pgrep java)

# Check connections
ss -ant | grep :5222/9090 | wc -l

# Monitor I/O
iotop -p $(pgrep java)
```

### Debug Mode

```bash
# Run in debug mode
sudo openfire -d
# or
sudo openfire debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  openfire:
    image: openfire:latest
    container_name: openfire
    ports:
      - "5222/9090:5222/9090"
    volumes:
      - ./config:/etc/openfire
      - ./data:/var/lib/openfire
    environment:
      - openfire_CONFIG=/etc/openfire/openfire.conf
    restart: unless-stopped
    networks:
      - openfire_net

networks:
  openfire_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openfire
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openfire
  template:
    metadata:
      labels:
        app: openfire
    spec:
      containers:
      - name: openfire
        image: openfire:latest
        ports:
        - containerPort: 5222/9090
        volumeMounts:
        - name: config
          mountPath: /etc/openfire
      volumes:
      - name: config
        configMap:
          name: openfire-config
---
apiVersion: v1
kind: Service
metadata:
  name: openfire
spec:
  selector:
    app: openfire
  ports:
  - port: 5222/9090
    targetPort: 5222/9090
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Openfire
  hosts: all
  become: yes
  tasks:
    - name: Install openfire
      package:
        name: openfire
        state: present
    
    - name: Configure openfire
      template:
        src: openfire.conf.j2
        dest: /etc/openfire/openfire.conf
        owner: openfire
        group: openfire
        mode: '0640'
      notify: restart openfire
    
    - name: Start and enable openfire
      systemd:
        name: openfire
        state: started
        enabled: yes
  
  handlers:
    - name: restart openfire
      systemd:
        name: openfire
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update openfire

# Debian/Ubuntu
sudo apt update && sudo apt upgrade openfire

# Arch Linux
sudo pacman -Syu openfire

# Alpine Linux
apk update && apk upgrade openfire

# openSUSE
sudo zypper update openfire

# FreeBSD
pkg update && pkg upgrade openfire

# Always backup before updates
tar -czf /backup/openfire-pre-update-$(date +%Y%m%d).tar.gz /etc/openfire

# Restart after updates
sudo systemctl restart openfire
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/openfire -name "*.log" -mtime +30 -delete

# Verify integrity
sudo openfire --verify || sudo openfire check

# Update databases (if applicable)
sudo openfire-update-db

# Optimize performance
sudo openfire-optimize

# Check for security updates
sudo openfire --security-check
```

## Additional Resources

- Official Documentation: https://docs.openfire.org/
- GitHub Repository: https://github.com/openfire/openfire
- Community Forum: https://forum.openfire.org/
- Wiki: https://wiki.openfire.org/
- Comparison vs ejabberd, Prosody, Tigase, Spark: https://docs.openfire.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
