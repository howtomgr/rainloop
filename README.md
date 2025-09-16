# RainLoop Installation Guide

RainLoop is a free and open-source Webmail. Simple, modern & fast web-based email client

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
  - Network: 80/443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default rainloop port)
- **Dependencies**:
  - php, php-curl, php-json
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

# Install rainloop
sudo dnf install -y rainloop php, php-curl, php-json

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo firewall-cmd --permanent --add-service=rainloop
sudo firewall-cmd --reload

# Verify installation
rainloop --version || systemctl status httpd
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rainloop
sudo apt install -y rainloop php, php-curl, php-json

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo ufw allow 80/443

# Verify installation
rainloop --version || systemctl status httpd
```

### Arch Linux

```bash
# Install rainloop
sudo pacman -S rainloop

# Enable and start service
sudo systemctl enable --now httpd

# Verify installation
rainloop --version || systemctl status httpd
```

### Alpine Linux

```bash
# Install rainloop
apk add --no-cache rainloop

# Enable and start service
rc-update add httpd default
rc-service httpd start

# Verify installation
rainloop --version || rc-service httpd status
```

### openSUSE/SLES

```bash
# Install rainloop
sudo zypper install -y rainloop php, php-curl, php-json

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo firewall-cmd --permanent --add-service=rainloop
sudo firewall-cmd --reload

# Verify installation
rainloop --version || systemctl status httpd
```

### macOS

```bash
# Using Homebrew
brew install rainloop

# Start service
brew services start rainloop

# Verify installation
rainloop --version
```

### FreeBSD

```bash
# Using pkg
pkg install rainloop

# Enable in rc.conf
echo 'httpd_enable="YES"' >> /etc/rc.conf

# Start service
service httpd start

# Verify installation
rainloop --version || service httpd status
```

### Windows

```powershell
# Using Chocolatey
choco install rainloop

# Or using Scoop
scoop install rainloop

# Verify installation
rainloop --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /var/www/rainloop/data

# Set up basic configuration
sudo tee /var/www/rainloop/data/rainloop.conf << 'EOF'
# RainLoop Configuration
cache_driver = APC
EOF

# Test configuration
sudo rainloop -t || sudo httpd configtest

# Reload service
sudo systemctl reload httpd
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R rainloop:rainloop /var/www/rainloop/data
sudo chmod 750 /var/www/rainloop/data

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable httpd

# Start service
sudo systemctl start httpd

# Stop service
sudo systemctl stop httpd

# Restart service
sudo systemctl restart httpd

# Reload configuration
sudo systemctl reload httpd

# Check status
sudo systemctl status httpd

# View logs
sudo journalctl -u httpd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add httpd default

# Start service
rc-service httpd start

# Stop service
rc-service httpd stop

# Restart service
rc-service httpd restart

# Check status
rc-service httpd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'httpd_enable="YES"' >> /etc/rc.conf

# Start service
service httpd start

# Stop service
service httpd stop

# Restart service
service httpd restart

# Check status
service httpd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rainloop
brew services stop rainloop
brew services restart rainloop

# Check status
brew services list | grep rainloop
```

### Windows Service Manager

```powershell
# Start service
net start httpd

# Stop service
net stop httpd

# Using PowerShell
Start-Service httpd
Stop-Service httpd
Restart-Service httpd

# Check status
Get-Service httpd
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /var/www/rainloop/data/rainloop.conf << 'EOF'
cache_driver = APC
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart httpd
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
upstream rainloop_backend {
    server 127.0.0.1:80/443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name rainloop.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rainloop.example.com;

    ssl_certificate /etc/ssl/certs/rainloop.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rainloop.example.com.key;

    location / {
        proxy_pass http://rainloop_backend;
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
    ServerName rainloop.example.com
    Redirect permanent / https://rainloop.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rainloop.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rainloop.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rainloop.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:80/443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rainloop_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rainloop.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rainloop_backend

backend rainloop_backend
    balance roundrobin
    option httpchk GET /health
    server rainloop1 127.0.0.1:80/443 check
    server rainloop2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rainloop:rainloop /var/www/rainloop/data
sudo chmod 750 /var/www/rainloop/data

# Configure firewall
sudo firewall-cmd --permanent --add-service=rainloop
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/rainloop.conf << 'EOF'
[rainloop]
enabled = true
port = 80/443
filter = rainloop
logpath = /var/log/rainloop/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/rainloop.key \
    -out /etc/ssl/certs/rainloop.crt

# Configure SSL in rainloop
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE rainloop_db;
CREATE USER rainloop_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE rainloop_db TO rainloop_user;
EOF

# Configure rainloop to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE rainloop_db;
CREATE USER 'rainloop_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON rainloop_db.* TO 'rainloop_user'@'localhost';
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

# RainLoop specific tuning
cache_driver = APC
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
rainloop soft nofile 65535
rainloop hard nofile 65535
rainloop soft nproc 32768
rainloop hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'rainloop'
    static_configs:
      - targets: ['localhost:80/443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet httpd; then
    echo "RainLoop is running"
    exit 0
else
    echo "RainLoop is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/rainloop << 'EOF'
/var/log/rainloop/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 rainloop rainloop
    postrotate
        systemctl reload httpd > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# RainLoop backup script
BACKUP_DIR="/backup/rainloop"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop httpd

# Backup configuration
tar -czf "$BACKUP_DIR/rainloop-config-$DATE.tar.gz" /var/www/rainloop/data

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/rainloop-data-$DATE.tar.gz" /var/lib/rainloop

# Start service
systemctl start httpd

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop httpd

# Restore configuration
sudo tar -xzf /backup/rainloop/rainloop-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/rainloop/rainloop-data-*.tar.gz -C /

# Set permissions
sudo chown -R rainloop:rainloop /var/www/rainloop/data
sudo chown -R rainloop:rainloop /var/lib/rainloop

# Start service
sudo systemctl start httpd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u httpd -n 100
sudo tail -f /var/log/rainloop/*.log

# Check configuration
sudo rainloop -t || sudo httpd configtest

# Check permissions
ls -la /var/www/rainloop/data
ls -la /var/lib/rainloop
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443
sudo netstat -tlnp | grep 80/443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 80/443
nc -zv localhost 80/443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep httpd)
htop -p $(pgrep httpd)

# Check connections
ss -ant | grep :80/443 | wc -l

# Monitor I/O
iotop -p $(pgrep httpd)
```

### Debug Mode

```bash
# Run in debug mode
sudo rainloop -d
# or
sudo httpd debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  rainloop:
    image: rainloop:latest
    container_name: rainloop
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/var/www/rainloop/data
      - ./data:/var/lib/rainloop
    environment:
      - rainloop_CONFIG=/var/www/rainloop/data/rainloop.conf
    restart: unless-stopped
    networks:
      - rainloop_net

networks:
  rainloop_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rainloop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rainloop
  template:
    metadata:
      labels:
        app: rainloop
    spec:
      containers:
      - name: rainloop
        image: rainloop:latest
        ports:
        - containerPort: 80/443
        volumeMounts:
        - name: config
          mountPath: /var/www/rainloop/data
      volumes:
      - name: config
        configMap:
          name: rainloop-config
---
apiVersion: v1
kind: Service
metadata:
  name: rainloop
spec:
  selector:
    app: rainloop
  ports:
  - port: 80/443
    targetPort: 80/443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure RainLoop
  hosts: all
  become: yes
  tasks:
    - name: Install rainloop
      package:
        name: rainloop
        state: present
    
    - name: Configure rainloop
      template:
        src: rainloop.conf.j2
        dest: /var/www/rainloop/data/rainloop.conf
        owner: rainloop
        group: rainloop
        mode: '0640'
      notify: restart rainloop
    
    - name: Start and enable rainloop
      systemd:
        name: httpd
        state: started
        enabled: yes
  
  handlers:
    - name: restart rainloop
      systemd:
        name: httpd
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rainloop

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rainloop

# Arch Linux
sudo pacman -Syu rainloop

# Alpine Linux
apk update && apk upgrade rainloop

# openSUSE
sudo zypper update rainloop

# FreeBSD
pkg update && pkg upgrade rainloop

# Always backup before updates
tar -czf /backup/rainloop-pre-update-$(date +%Y%m%d).tar.gz /var/www/rainloop/data

# Restart after updates
sudo systemctl restart httpd
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/rainloop -name "*.log" -mtime +30 -delete

# Verify integrity
sudo rainloop --verify || sudo httpd check

# Update databases (if applicable)
sudo rainloop-update-db

# Optimize performance
sudo rainloop-optimize

# Check for security updates
sudo rainloop --security-check
```

## Additional Resources

- Official Documentation: https://docs.rainloop.org/
- GitHub Repository: https://github.com/rainloop/rainloop
- Community Forum: https://forum.rainloop.org/
- Wiki: https://wiki.rainloop.org/
- Comparison vs Roundcube, SquirrelMail, Horde, AfterLogic: https://docs.rainloop.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
