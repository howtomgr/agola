# agola Installation Guide

agola is a free and open-source CI/CD system. Agola provides open source CI/CD system with advanced workflows

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
  - RAM: 2GB minimum
  - Storage: 20GB for runs
  - Network: HTTP/Git access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8000 (default agola port)
  - None
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

# Install agola
sudo dnf install -y agola

# Enable and start service
sudo systemctl enable --now agola

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
agola --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install agola
sudo apt install -y agola

# Enable and start service
sudo systemctl enable --now agola

# Configure firewall
sudo ufw allow 8000

# Verify installation
agola --version
```

### Arch Linux

```bash
# Install agola
sudo pacman -S agola

# Enable and start service
sudo systemctl enable --now agola

# Verify installation
agola --version
```

### Alpine Linux

```bash
# Install agola
apk add --no-cache agola

# Enable and start service
rc-update add agola default
rc-service agola start

# Verify installation
agola --version
```

### openSUSE/SLES

```bash
# Install agola
sudo zypper install -y agola

# Enable and start service
sudo systemctl enable --now agola

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
agola --version
```

### macOS

```bash
# Using Homebrew
brew install agola

# Start service
brew services start agola

# Verify installation
agola --version
```

### FreeBSD

```bash
# Using pkg
pkg install agola

# Enable in rc.conf
echo 'agola_enable="YES"' >> /etc/rc.conf

# Start service
service agola start

# Verify installation
agola --version
```

### Windows

```bash
# Using Chocolatey
choco install agola

# Or using Scoop
scoop install agola

# Verify installation
agola --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/agola

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
agola --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable agola

# Start service
sudo systemctl start agola

# Stop service
sudo systemctl stop agola

# Restart service
sudo systemctl restart agola

# Check status
sudo systemctl status agola

# View logs
sudo journalctl -u agola -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add agola default

# Start service
rc-service agola start

# Stop service
rc-service agola stop

# Restart service
rc-service agola restart

# Check status
rc-service agola status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'agola_enable="YES"' >> /etc/rc.conf

# Start service
service agola start

# Stop service
service agola stop

# Restart service
service agola restart

# Check status
service agola status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start agola
brew services stop agola
brew services restart agola

# Check status
brew services list | grep agola
```

### Windows Service Manager

```powershell
# Start service
net start agola

# Stop service
net stop agola

# Using PowerShell
Start-Service agola
Stop-Service agola
Restart-Service agola

# Check status
Get-Service agola
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream agola_backend {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name agola.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name agola.example.com;

    ssl_certificate /etc/ssl/certs/agola.example.com.crt;
    ssl_certificate_key /etc/ssl/private/agola.example.com.key;

    location / {
        proxy_pass http://agola_backend;
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
    ServerName agola.example.com
    Redirect permanent / https://agola.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName agola.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/agola.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/agola.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend agola_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/agola.pem
    redirect scheme https if !{ ssl_fc }
    default_backend agola_backend

backend agola_backend
    balance roundrobin
    server agola1 127.0.0.1:8000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R agola:agola /etc/agola
sudo chmod 750 /etc/agola

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
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
sudo systemctl status agola

# View logs
sudo journalctl -u agola -f

# Monitor resource usage
top -p $(pgrep agola)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/agola"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/agola-backup-$DATE.tar.gz" /etc/agola /var/lib/agola

echo "Backup completed: $BACKUP_DIR/agola-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop agola

# Restore from backup
tar -xzf /backup/agola/agola-backup-*.tar.gz -C /

# Start service
sudo systemctl start agola
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u agola -n 100
sudo tail -f /var/log/agola/agola.log

# Check configuration
agola --version

# Check permissions
ls -la /etc/agola
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8000

# Test connectivity
telnet localhost 8000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep agola)

# Check disk I/O
iotop -p $(pgrep agola)

# Check connections
ss -an | grep 8000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  agola:
    image: agola:latest
    ports:
      - "8000:8000"
    volumes:
      - ./config:/etc/agola
      - ./data:/var/lib/agola
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update agola

# Debian/Ubuntu
sudo apt update && sudo apt upgrade agola

# Arch Linux
sudo pacman -Syu agola

# Alpine Linux
apk update && apk upgrade agola

# openSUSE
sudo zypper update agola

# FreeBSD
pkg update && pkg upgrade agola

# Always backup before updates
tar -czf /backup/agola-pre-update-$(date +%Y%m%d).tar.gz /etc/agola

# Restart after updates
sudo systemctl restart agola
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/agola

# Clean old logs
find /var/log/agola -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/agola
```

## Additional Resources

- Official Documentation: https://docs.agola.org/
- GitHub Repository: https://github.com/agola/agola
- Community Forum: https://forum.agola.org/
- Best Practices Guide: https://docs.agola.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
