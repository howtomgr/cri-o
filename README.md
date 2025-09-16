# cri-o Installation Guide

cri-o is a free and open-source lightweight container runtime for Kubernetes. CRI-O provides a minimal runtime specifically for Kubernetes, serving as an alternative to Docker or containerd

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for installation
  - Network: Container networking
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 10010 (default cri-o port)
  - Unix socket based
- **Dependencies**:
  - See official documentation for specific requirements
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

# Install cri-o
sudo dnf install -y cri-o

# Enable and start service
sudo systemctl enable --now crio

# Configure firewall
sudo firewall-cmd --permanent --add-port=10010/tcp
sudo firewall-cmd --reload

# Verify installation
crio --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install cri-o
sudo apt install -y cri-o

# Enable and start service
sudo systemctl enable --now crio

# Configure firewall
sudo ufw allow 10010

# Verify installation
crio --version
```

### Arch Linux

```bash
# Install cri-o
sudo pacman -S cri-o

# Enable and start service
sudo systemctl enable --now crio

# Verify installation
crio --version
```

### Alpine Linux

```bash
# Install cri-o
apk add --no-cache cri-o

# Enable and start service
rc-update add crio default
rc-service crio start

# Verify installation
crio --version
```

### openSUSE/SLES

```bash
# Install cri-o
sudo zypper install -y cri-o

# Enable and start service
sudo systemctl enable --now crio

# Configure firewall
sudo firewall-cmd --permanent --add-port=10010/tcp
sudo firewall-cmd --reload

# Verify installation
crio --version
```

### macOS

```bash
# Using Homebrew
brew install cri-o

# Start service
brew services start cri-o

# Verify installation
crio --version
```

### FreeBSD

```bash
# Using pkg
pkg install cri-o

# Enable in rc.conf
echo 'crio_enable="YES"' >> /etc/rc.conf

# Start service
service crio start

# Verify installation
crio --version
```

### Windows

```bash
# Using Chocolatey
choco install cri-o

# Or using Scoop
scoop install cri-o

# Verify installation
crio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/cri-o

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
crio --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable crio

# Start service
sudo systemctl start crio

# Stop service
sudo systemctl stop crio

# Restart service
sudo systemctl restart crio

# Check status
sudo systemctl status crio

# View logs
sudo journalctl -u crio -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add crio default

# Start service
rc-service crio start

# Stop service
rc-service crio stop

# Restart service
rc-service crio restart

# Check status
rc-service crio status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'crio_enable="YES"' >> /etc/rc.conf

# Start service
service crio start

# Stop service
service crio stop

# Restart service
service crio restart

# Check status
service crio status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start cri-o
brew services stop cri-o
brew services restart cri-o

# Check status
brew services list | grep cri-o
```

### Windows Service Manager

```powershell
# Start service
net start crio

# Stop service
net stop crio

# Using PowerShell
Start-Service crio
Stop-Service crio
Restart-Service crio

# Check status
Get-Service crio
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream cri-o_backend {
    server 127.0.0.1:10010;
}

server {
    listen 80;
    server_name cri-o.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cri-o.example.com;

    ssl_certificate /etc/ssl/certs/cri-o.example.com.crt;
    ssl_certificate_key /etc/ssl/private/cri-o.example.com.key;

    location / {
        proxy_pass http://cri-o_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName cri-o.example.com
    Redirect permanent / https://cri-o.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName cri-o.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cri-o.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/cri-o.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:10010/
    ProxyPassReverse / http://127.0.0.1:10010/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend cri-o_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/cri-o.pem
    redirect scheme https if !{ ssl_fc }
    default_backend cri-o_backend

backend cri-o_backend
    balance roundrobin
    server cri-o1 127.0.0.1:10010 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R cri-o:cri-o /etc/cri-o
sudo chmod 750 /etc/cri-o

# Configure firewall
sudo firewall-cmd --permanent --add-port=10010/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status crio

# View logs
sudo journalctl -u crio -f

# Monitor resource usage
top -p $(pgrep cri-o)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/cri-o"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/cri-o-backup-$DATE.tar.gz" /etc/cri-o /var/lib/cri-o

echo "Backup completed: $BACKUP_DIR/cri-o-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop crio

# Restore from backup
tar -xzf /backup/cri-o/cri-o-backup-*.tar.gz -C /

# Start service
sudo systemctl start crio
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u crio -n 100
sudo tail -f /var/log/cri-o/cri-o.log

# Check configuration
crio --version

# Check permissions
ls -la /etc/cri-o
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 10010

# Test connectivity
telnet localhost 10010

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep cri-o)

# Check disk I/O
iotop -p $(pgrep cri-o)

# Check connections
ss -an | grep 10010
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  cri-o:
    image: cri-o:latest
    ports:
      - "10010:10010"
    volumes:
      - ./config:/etc/cri-o
      - ./data:/var/lib/cri-o
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update cri-o

# Debian/Ubuntu
sudo apt update && sudo apt upgrade cri-o

# Arch Linux
sudo pacman -Syu cri-o

# Alpine Linux
apk update && apk upgrade cri-o

# openSUSE
sudo zypper update cri-o

# FreeBSD
pkg update && pkg upgrade cri-o

# Always backup before updates
tar -czf /backup/cri-o-pre-update-$(date +%Y%m%d).tar.gz /etc/cri-o

# Restart after updates
sudo systemctl restart crio
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/cri-o

# Clean old logs
find /var/log/cri-o -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/cri-o
```

## Additional Resources

- Official Documentation: https://docs.cri-o.org/
- GitHub Repository: https://github.com/cri-o/cri-o
- Community Forum: https://forum.cri-o.org/
- Best Practices Guide: https://docs.cri-o.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
