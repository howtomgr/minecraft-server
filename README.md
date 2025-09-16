# minecraft-server Installation Guide

minecraft-server is a free and open-source Minecraft game server. Official Minecraft Java Edition server for hosting multiplayer worlds

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
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for worlds
  - Network: Game protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 25565 (default minecraft-server port)
  - RCON on 25575
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

# Install minecraft-server
sudo dnf install -y minecraft-server

# Enable and start service
sudo systemctl enable --now minecraft

# Configure firewall
sudo firewall-cmd --permanent --add-port=25565/tcp
sudo firewall-cmd --reload

# Verify installation
java -jar minecraft_server.jar --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install minecraft-server
sudo apt install -y minecraft-server

# Enable and start service
sudo systemctl enable --now minecraft

# Configure firewall
sudo ufw allow 25565

# Verify installation
java -jar minecraft_server.jar --version
```

### Arch Linux

```bash
# Install minecraft-server
sudo pacman -S minecraft-server

# Enable and start service
sudo systemctl enable --now minecraft

# Verify installation
java -jar minecraft_server.jar --version
```

### Alpine Linux

```bash
# Install minecraft-server
apk add --no-cache minecraft-server

# Enable and start service
rc-update add minecraft default
rc-service minecraft start

# Verify installation
java -jar minecraft_server.jar --version
```

### openSUSE/SLES

```bash
# Install minecraft-server
sudo zypper install -y minecraft-server

# Enable and start service
sudo systemctl enable --now minecraft

# Configure firewall
sudo firewall-cmd --permanent --add-port=25565/tcp
sudo firewall-cmd --reload

# Verify installation
java -jar minecraft_server.jar --version
```

### macOS

```bash
# Using Homebrew
brew install minecraft-server

# Start service
brew services start minecraft-server

# Verify installation
java -jar minecraft_server.jar --version
```

### FreeBSD

```bash
# Using pkg
pkg install minecraft-server

# Enable in rc.conf
echo 'minecraft_enable="YES"' >> /etc/rc.conf

# Start service
service minecraft start

# Verify installation
java -jar minecraft_server.jar --version
```

### Windows

```bash
# Using Chocolatey
choco install minecraft-server

# Or using Scoop
scoop install minecraft-server

# Verify installation
java -jar minecraft_server.jar --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/minecraft-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
java -jar minecraft_server.jar --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable minecraft

# Start service
sudo systemctl start minecraft

# Stop service
sudo systemctl stop minecraft

# Restart service
sudo systemctl restart minecraft

# Check status
sudo systemctl status minecraft

# View logs
sudo journalctl -u minecraft -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add minecraft default

# Start service
rc-service minecraft start

# Stop service
rc-service minecraft stop

# Restart service
rc-service minecraft restart

# Check status
rc-service minecraft status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'minecraft_enable="YES"' >> /etc/rc.conf

# Start service
service minecraft start

# Stop service
service minecraft stop

# Restart service
service minecraft restart

# Check status
service minecraft status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start minecraft-server
brew services stop minecraft-server
brew services restart minecraft-server

# Check status
brew services list | grep minecraft-server
```

### Windows Service Manager

```powershell
# Start service
net start minecraft

# Stop service
net stop minecraft

# Using PowerShell
Start-Service minecraft
Stop-Service minecraft
Restart-Service minecraft

# Check status
Get-Service minecraft
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream minecraft-server_backend {
    server 127.0.0.1:25565;
}

server {
    listen 80;
    server_name minecraft-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name minecraft-server.example.com;

    ssl_certificate /etc/ssl/certs/minecraft-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/minecraft-server.example.com.key;

    location / {
        proxy_pass http://minecraft-server_backend;
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
    ServerName minecraft-server.example.com
    Redirect permanent / https://minecraft-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName minecraft-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/minecraft-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/minecraft-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:25565/
    ProxyPassReverse / http://127.0.0.1:25565/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend minecraft-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/minecraft-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend minecraft-server_backend

backend minecraft-server_backend
    balance roundrobin
    server minecraft-server1 127.0.0.1:25565 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R minecraft-server:minecraft-server /etc/minecraft-server
sudo chmod 750 /etc/minecraft-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=25565/tcp
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
sudo systemctl status minecraft

# View logs
sudo journalctl -u minecraft -f

# Monitor resource usage
top -p $(pgrep minecraft-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/minecraft-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/minecraft-server-backup-$DATE.tar.gz" /etc/minecraft-server /var/lib/minecraft-server

echo "Backup completed: $BACKUP_DIR/minecraft-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop minecraft

# Restore from backup
tar -xzf /backup/minecraft-server/minecraft-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start minecraft
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u minecraft -n 100
sudo tail -f /var/log/minecraft-server/minecraft-server.log

# Check configuration
java -jar minecraft_server.jar --version

# Check permissions
ls -la /etc/minecraft-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 25565

# Test connectivity
telnet localhost 25565

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep minecraft-server)

# Check disk I/O
iotop -p $(pgrep minecraft-server)

# Check connections
ss -an | grep 25565
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  minecraft-server:
    image: minecraft-server:latest
    ports:
      - "25565:25565"
    volumes:
      - ./config:/etc/minecraft-server
      - ./data:/var/lib/minecraft-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update minecraft-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade minecraft-server

# Arch Linux
sudo pacman -Syu minecraft-server

# Alpine Linux
apk update && apk upgrade minecraft-server

# openSUSE
sudo zypper update minecraft-server

# FreeBSD
pkg update && pkg upgrade minecraft-server

# Always backup before updates
tar -czf /backup/minecraft-server-pre-update-$(date +%Y%m%d).tar.gz /etc/minecraft-server

# Restart after updates
sudo systemctl restart minecraft
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/minecraft-server

# Clean old logs
find /var/log/minecraft-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/minecraft-server
```

## Additional Resources

- Official Documentation: https://docs.minecraft-server.org/
- GitHub Repository: https://github.com/minecraft-server/minecraft-server
- Community Forum: https://forum.minecraft-server.org/
- Best Practices Guide: https://docs.minecraft-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
