# temporal Installation Guide

temporal is a free and open-source workflow orchestration. Temporal provides workflow orchestration platform for mission-critical applications

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
  - CPU: 4+ cores
  - RAM: 4GB minimum
  - Storage: 20GB for history
  - Network: gRPC/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 7233 (default temporal port)
  - Web UI on 8088
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

# Install temporal
sudo dnf install -y temporal

# Enable and start service
sudo systemctl enable --now temporal

# Configure firewall
sudo firewall-cmd --permanent --add-port=7233/tcp
sudo firewall-cmd --reload

# Verify installation
temporal --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install temporal
sudo apt install -y temporal

# Enable and start service
sudo systemctl enable --now temporal

# Configure firewall
sudo ufw allow 7233

# Verify installation
temporal --version
```

### Arch Linux

```bash
# Install temporal
sudo pacman -S temporal

# Enable and start service
sudo systemctl enable --now temporal

# Verify installation
temporal --version
```

### Alpine Linux

```bash
# Install temporal
apk add --no-cache temporal

# Enable and start service
rc-update add temporal default
rc-service temporal start

# Verify installation
temporal --version
```

### openSUSE/SLES

```bash
# Install temporal
sudo zypper install -y temporal

# Enable and start service
sudo systemctl enable --now temporal

# Configure firewall
sudo firewall-cmd --permanent --add-port=7233/tcp
sudo firewall-cmd --reload

# Verify installation
temporal --version
```

### macOS

```bash
# Using Homebrew
brew install temporal

# Start service
brew services start temporal

# Verify installation
temporal --version
```

### FreeBSD

```bash
# Using pkg
pkg install temporal

# Enable in rc.conf
echo 'temporal_enable="YES"' >> /etc/rc.conf

# Start service
service temporal start

# Verify installation
temporal --version
```

### Windows

```bash
# Using Chocolatey
choco install temporal

# Or using Scoop
scoop install temporal

# Verify installation
temporal --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/temporal

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
temporal --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable temporal

# Start service
sudo systemctl start temporal

# Stop service
sudo systemctl stop temporal

# Restart service
sudo systemctl restart temporal

# Check status
sudo systemctl status temporal

# View logs
sudo journalctl -u temporal -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add temporal default

# Start service
rc-service temporal start

# Stop service
rc-service temporal stop

# Restart service
rc-service temporal restart

# Check status
rc-service temporal status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'temporal_enable="YES"' >> /etc/rc.conf

# Start service
service temporal start

# Stop service
service temporal stop

# Restart service
service temporal restart

# Check status
service temporal status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start temporal
brew services stop temporal
brew services restart temporal

# Check status
brew services list | grep temporal
```

### Windows Service Manager

```powershell
# Start service
net start temporal

# Stop service
net stop temporal

# Using PowerShell
Start-Service temporal
Stop-Service temporal
Restart-Service temporal

# Check status
Get-Service temporal
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream temporal_backend {
    server 127.0.0.1:7233;
}

server {
    listen 80;
    server_name temporal.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name temporal.example.com;

    ssl_certificate /etc/ssl/certs/temporal.example.com.crt;
    ssl_certificate_key /etc/ssl/private/temporal.example.com.key;

    location / {
        proxy_pass http://temporal_backend;
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
    ServerName temporal.example.com
    Redirect permanent / https://temporal.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName temporal.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/temporal.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/temporal.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:7233/
    ProxyPassReverse / http://127.0.0.1:7233/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend temporal_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/temporal.pem
    redirect scheme https if !{ ssl_fc }
    default_backend temporal_backend

backend temporal_backend
    balance roundrobin
    server temporal1 127.0.0.1:7233 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R temporal:temporal /etc/temporal
sudo chmod 750 /etc/temporal

# Configure firewall
sudo firewall-cmd --permanent --add-port=7233/tcp
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
sudo systemctl status temporal

# View logs
sudo journalctl -u temporal -f

# Monitor resource usage
top -p $(pgrep temporal)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/temporal"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/temporal-backup-$DATE.tar.gz" /etc/temporal /var/lib/temporal

echo "Backup completed: $BACKUP_DIR/temporal-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop temporal

# Restore from backup
tar -xzf /backup/temporal/temporal-backup-*.tar.gz -C /

# Start service
sudo systemctl start temporal
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u temporal -n 100
sudo tail -f /var/log/temporal/temporal.log

# Check configuration
temporal --version

# Check permissions
ls -la /etc/temporal
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 7233

# Test connectivity
telnet localhost 7233

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep temporal)

# Check disk I/O
iotop -p $(pgrep temporal)

# Check connections
ss -an | grep 7233
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  temporal:
    image: temporal:latest
    ports:
      - "7233:7233"
    volumes:
      - ./config:/etc/temporal
      - ./data:/var/lib/temporal
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update temporal

# Debian/Ubuntu
sudo apt update && sudo apt upgrade temporal

# Arch Linux
sudo pacman -Syu temporal

# Alpine Linux
apk update && apk upgrade temporal

# openSUSE
sudo zypper update temporal

# FreeBSD
pkg update && pkg upgrade temporal

# Always backup before updates
tar -czf /backup/temporal-pre-update-$(date +%Y%m%d).tar.gz /etc/temporal

# Restart after updates
sudo systemctl restart temporal
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/temporal

# Clean old logs
find /var/log/temporal -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/temporal
```

## Additional Resources

- Official Documentation: https://docs.temporal.org/
- GitHub Repository: https://github.com/temporal/temporal
- Community Forum: https://forum.temporal.org/
- Best Practices Guide: https://docs.temporal.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
