# kallithea Installation Guide

kallithea is a free and open-source source code management. Kallithea provides source code management for Git and Mercurial

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
  - RAM: 1GB minimum
  - Storage: 5GB for repos
  - Network: HTTP/SSH access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5000 (default kallithea port)
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

# Install kallithea
sudo dnf install -y kallithea

# Enable and start service
sudo systemctl enable --now kallithea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
kallithea --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kallithea
sudo apt install -y kallithea

# Enable and start service
sudo systemctl enable --now kallithea

# Configure firewall
sudo ufw allow 5000

# Verify installation
kallithea --version
```

### Arch Linux

```bash
# Install kallithea
sudo pacman -S kallithea

# Enable and start service
sudo systemctl enable --now kallithea

# Verify installation
kallithea --version
```

### Alpine Linux

```bash
# Install kallithea
apk add --no-cache kallithea

# Enable and start service
rc-update add kallithea default
rc-service kallithea start

# Verify installation
kallithea --version
```

### openSUSE/SLES

```bash
# Install kallithea
sudo zypper install -y kallithea

# Enable and start service
sudo systemctl enable --now kallithea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
kallithea --version
```

### macOS

```bash
# Using Homebrew
brew install kallithea

# Start service
brew services start kallithea

# Verify installation
kallithea --version
```

### FreeBSD

```bash
# Using pkg
pkg install kallithea

# Enable in rc.conf
echo 'kallithea_enable="YES"' >> /etc/rc.conf

# Start service
service kallithea start

# Verify installation
kallithea --version
```

### Windows

```bash
# Using Chocolatey
choco install kallithea

# Or using Scoop
scoop install kallithea

# Verify installation
kallithea --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kallithea

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kallithea --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kallithea

# Start service
sudo systemctl start kallithea

# Stop service
sudo systemctl stop kallithea

# Restart service
sudo systemctl restart kallithea

# Check status
sudo systemctl status kallithea

# View logs
sudo journalctl -u kallithea -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kallithea default

# Start service
rc-service kallithea start

# Stop service
rc-service kallithea stop

# Restart service
rc-service kallithea restart

# Check status
rc-service kallithea status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kallithea_enable="YES"' >> /etc/rc.conf

# Start service
service kallithea start

# Stop service
service kallithea stop

# Restart service
service kallithea restart

# Check status
service kallithea status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kallithea
brew services stop kallithea
brew services restart kallithea

# Check status
brew services list | grep kallithea
```

### Windows Service Manager

```powershell
# Start service
net start kallithea

# Stop service
net stop kallithea

# Using PowerShell
Start-Service kallithea
Stop-Service kallithea
Restart-Service kallithea

# Check status
Get-Service kallithea
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kallithea_backend {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name kallithea.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kallithea.example.com;

    ssl_certificate /etc/ssl/certs/kallithea.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kallithea.example.com.key;

    location / {
        proxy_pass http://kallithea_backend;
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
    ServerName kallithea.example.com
    Redirect permanent / https://kallithea.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kallithea.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kallithea.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kallithea.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kallithea_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kallithea.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kallithea_backend

backend kallithea_backend
    balance roundrobin
    server kallithea1 127.0.0.1:5000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kallithea:kallithea /etc/kallithea
sudo chmod 750 /etc/kallithea

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
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
sudo systemctl status kallithea

# View logs
sudo journalctl -u kallithea -f

# Monitor resource usage
top -p $(pgrep kallithea)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kallithea"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kallithea-backup-$DATE.tar.gz" /etc/kallithea /var/lib/kallithea

echo "Backup completed: $BACKUP_DIR/kallithea-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kallithea

# Restore from backup
tar -xzf /backup/kallithea/kallithea-backup-*.tar.gz -C /

# Start service
sudo systemctl start kallithea
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kallithea -n 100
sudo tail -f /var/log/kallithea/kallithea.log

# Check configuration
kallithea --version

# Check permissions
ls -la /etc/kallithea
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5000

# Test connectivity
telnet localhost 5000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kallithea)

# Check disk I/O
iotop -p $(pgrep kallithea)

# Check connections
ss -an | grep 5000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kallithea:
    image: kallithea:latest
    ports:
      - "5000:5000"
    volumes:
      - ./config:/etc/kallithea
      - ./data:/var/lib/kallithea
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kallithea

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kallithea

# Arch Linux
sudo pacman -Syu kallithea

# Alpine Linux
apk update && apk upgrade kallithea

# openSUSE
sudo zypper update kallithea

# FreeBSD
pkg update && pkg upgrade kallithea

# Always backup before updates
tar -czf /backup/kallithea-pre-update-$(date +%Y%m%d).tar.gz /etc/kallithea

# Restart after updates
sudo systemctl restart kallithea
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kallithea

# Clean old logs
find /var/log/kallithea -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kallithea
```

## Additional Resources

- Official Documentation: https://docs.kallithea.org/
- GitHub Repository: https://github.com/kallithea/kallithea
- Community Forum: https://forum.kallithea.org/
- Best Practices Guide: https://docs.kallithea.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
