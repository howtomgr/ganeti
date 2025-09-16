# ganeti Installation Guide

ganeti is a free and open-source cluster virtual server management. Ganeti manages clusters of virtual machines built on Xen or KVM

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
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 20GB for system
  - Network: Cluster network
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1811 (default ganeti port)
  - DRBD ports
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

# Install ganeti
sudo dnf install -y ganeti

# Enable and start service
sudo systemctl enable --now ganeti

# Configure firewall
sudo firewall-cmd --permanent --add-port=1811/tcp
sudo firewall-cmd --reload

# Verify installation
ganeti --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ganeti
sudo apt install -y ganeti

# Enable and start service
sudo systemctl enable --now ganeti

# Configure firewall
sudo ufw allow 1811

# Verify installation
ganeti --version
```

### Arch Linux

```bash
# Install ganeti
sudo pacman -S ganeti

# Enable and start service
sudo systemctl enable --now ganeti

# Verify installation
ganeti --version
```

### Alpine Linux

```bash
# Install ganeti
apk add --no-cache ganeti

# Enable and start service
rc-update add ganeti default
rc-service ganeti start

# Verify installation
ganeti --version
```

### openSUSE/SLES

```bash
# Install ganeti
sudo zypper install -y ganeti

# Enable and start service
sudo systemctl enable --now ganeti

# Configure firewall
sudo firewall-cmd --permanent --add-port=1811/tcp
sudo firewall-cmd --reload

# Verify installation
ganeti --version
```

### macOS

```bash
# Using Homebrew
brew install ganeti

# Start service
brew services start ganeti

# Verify installation
ganeti --version
```

### FreeBSD

```bash
# Using pkg
pkg install ganeti

# Enable in rc.conf
echo 'ganeti_enable="YES"' >> /etc/rc.conf

# Start service
service ganeti start

# Verify installation
ganeti --version
```

### Windows

```bash
# Using Chocolatey
choco install ganeti

# Or using Scoop
scoop install ganeti

# Verify installation
ganeti --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ganeti

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ganeti --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ganeti

# Start service
sudo systemctl start ganeti

# Stop service
sudo systemctl stop ganeti

# Restart service
sudo systemctl restart ganeti

# Check status
sudo systemctl status ganeti

# View logs
sudo journalctl -u ganeti -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ganeti default

# Start service
rc-service ganeti start

# Stop service
rc-service ganeti stop

# Restart service
rc-service ganeti restart

# Check status
rc-service ganeti status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ganeti_enable="YES"' >> /etc/rc.conf

# Start service
service ganeti start

# Stop service
service ganeti stop

# Restart service
service ganeti restart

# Check status
service ganeti status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ganeti
brew services stop ganeti
brew services restart ganeti

# Check status
brew services list | grep ganeti
```

### Windows Service Manager

```powershell
# Start service
net start ganeti

# Stop service
net stop ganeti

# Using PowerShell
Start-Service ganeti
Stop-Service ganeti
Restart-Service ganeti

# Check status
Get-Service ganeti
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ganeti_backend {
    server 127.0.0.1:1811;
}

server {
    listen 80;
    server_name ganeti.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ganeti.example.com;

    ssl_certificate /etc/ssl/certs/ganeti.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ganeti.example.com.key;

    location / {
        proxy_pass http://ganeti_backend;
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
    ServerName ganeti.example.com
    Redirect permanent / https://ganeti.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ganeti.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ganeti.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ganeti.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1811/
    ProxyPassReverse / http://127.0.0.1:1811/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ganeti_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ganeti.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ganeti_backend

backend ganeti_backend
    balance roundrobin
    server ganeti1 127.0.0.1:1811 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ganeti:ganeti /etc/ganeti
sudo chmod 750 /etc/ganeti

# Configure firewall
sudo firewall-cmd --permanent --add-port=1811/tcp
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
sudo systemctl status ganeti

# View logs
sudo journalctl -u ganeti -f

# Monitor resource usage
top -p $(pgrep ganeti)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ganeti"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ganeti-backup-$DATE.tar.gz" /etc/ganeti /var/lib/ganeti

echo "Backup completed: $BACKUP_DIR/ganeti-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ganeti

# Restore from backup
tar -xzf /backup/ganeti/ganeti-backup-*.tar.gz -C /

# Start service
sudo systemctl start ganeti
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ganeti -n 100
sudo tail -f /var/log/ganeti/ganeti.log

# Check configuration
ganeti --version

# Check permissions
ls -la /etc/ganeti
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1811

# Test connectivity
telnet localhost 1811

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ganeti)

# Check disk I/O
iotop -p $(pgrep ganeti)

# Check connections
ss -an | grep 1811
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ganeti:
    image: ganeti:latest
    ports:
      - "1811:1811"
    volumes:
      - ./config:/etc/ganeti
      - ./data:/var/lib/ganeti
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ganeti

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ganeti

# Arch Linux
sudo pacman -Syu ganeti

# Alpine Linux
apk update && apk upgrade ganeti

# openSUSE
sudo zypper update ganeti

# FreeBSD
pkg update && pkg upgrade ganeti

# Always backup before updates
tar -czf /backup/ganeti-pre-update-$(date +%Y%m%d).tar.gz /etc/ganeti

# Restart after updates
sudo systemctl restart ganeti
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ganeti

# Clean old logs
find /var/log/ganeti -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ganeti
```

## Additional Resources

- Official Documentation: https://docs.ganeti.org/
- GitHub Repository: https://github.com/ganeti/ganeti
- Community Forum: https://forum.ganeti.org/
- Best Practices Guide: https://docs.ganeti.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
