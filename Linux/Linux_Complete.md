# Linux Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Distribution Differences](#distribution-differences)
- [Package Management](#package-management)
- [System Information](#system-information)
- [User & Group Management](#user--group-management)
- [File & Directory Management](#file--directory-management)
- [Permissions & Ownership](#permissions--ownership)
- [Process Management](#process-management)
- [Service Management](#service-management)
- [Network Configuration](#network-configuration)
- [Disk & Storage Management](#disk--storage-management)
- [SSH & Remote Access](#ssh--remote-access)
- [Firewall Configuration](#firewall-configuration)
- [Performance Monitoring](#performance-monitoring)
- [Log Management](#log-management)
- [Cron Jobs & Scheduling](#cron-jobs--scheduling)
- [Shell Scripting](#shell-scripting)
- [Text Processing](#text-processing)
- [Archive & Compression](#archive--compression)
- [Security Hardening](#security-hardening)
- [Troubleshooting](#troubleshooting)
- [Advanced Topics](#advanced-topics)

---

## Introduction

Linux is a family of open-source Unix-like operating systems based on the Linux kernel. This guide covers the three most popular enterprise distributions:

- **Ubuntu/Debian**: Debian-based distributions using APT package manager
- **CentOS/RHEL/Rocky/AlmaLinux**: Red Hat-based distributions using YUM/DNF
- **Common Commands**: Universal commands that work across all distributions

---

## Distribution Differences

### Quick Reference

| Feature | Ubuntu/Debian | CentOS/RHEL/Rocky |
|---------|---------------|-------------------|
| Package Manager | APT (apt, apt-get) | YUM/DNF |
| Package Format | .deb | .rpm |
| Service Manager | systemd | systemd |
| Firewall | ufw/iptables | firewalld/iptables |
| Init System | systemd | systemd |
| Default Shell | bash | bash |
| Config Location | /etc/network/ | /etc/sysconfig/network-scripts/ |

### Version Identification

```bash
# Check distribution
cat /etc/os-release
lsb_release -a

# Check kernel version
uname -r
uname -a

# Check architecture
uname -m
arch

# Check hostname
hostname
hostnamectl

# Check system uptime
uptime
```

---

## Package Management

### APT (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Upgrade all packages
sudo apt upgrade

# Full upgrade (handle dependencies)
sudo apt full-upgrade

# Dist upgrade (for version upgrades)
sudo apt dist-upgrade

# Install package
sudo apt install nginx
sudo apt install nginx mysql-server php

# Install specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Remove package
sudo apt remove nginx

# Remove package and config files
sudo apt purge nginx

# Autoremove unused dependencies
sudo apt autoremove

# Clean package cache
sudo apt clean
sudo apt autoclean

# Search for package
apt search nginx
apt-cache search nginx

# Show package information
apt show nginx
apt-cache show nginx

# List installed packages
apt list --installed
dpkg -l

# List upgradable packages
apt list --upgradable

# Show package dependencies
apt-cache depends nginx

# Show reverse dependencies
apt-cache rdepends nginx

# Download package without installing
apt download nginx

# Reinstall package
sudo apt reinstall nginx

# Hold package version (prevent upgrade)
sudo apt-mark hold nginx

# Unhold package
sudo apt-mark unhold nginx

# Add repository
sudo add-apt-repository ppa:nginx/stable
sudo apt update

# Remove repository
sudo add-apt-repository --remove ppa:nginx/stable

# Edit sources list
sudo nano /etc/apt/sources.list
sudo nano /etc/apt/sources.list.d/custom.list

# Fix broken dependencies
sudo apt --fix-broken install
sudo dpkg --configure -a
```

### YUM/DNF (CentOS/RHEL/Rocky)

```bash
# Update package index
sudo yum check-update
sudo dnf check-update

# Update all packages
sudo yum update
sudo dnf update

# Install package
sudo yum install nginx
sudo dnf install nginx

# Install specific version
sudo yum install nginx-1.20.1

# Remove package
sudo yum remove nginx
sudo dnf remove nginx

# Remove package and dependencies
sudo yum autoremove nginx

# Search for package
yum search nginx
dnf search nginx

# Show package information
yum info nginx
dnf info nginx

# List installed packages
yum list installed
dnf list installed
rpm -qa

# List available packages
yum list available
dnf list available

# Show package dependencies
yum deplist nginx
dnf repoquery --requires nginx

# Download package without installing
yumdownloader nginx
dnf download nginx

# Reinstall package
sudo yum reinstall nginx

# Clean cache
sudo yum clean all
sudo dnf clean all

# Enable repository
sudo yum-config-manager --enable epel
sudo dnf config-manager --set-enabled epel

# Disable repository
sudo yum-config-manager --disable epel

# Add repository (EPEL)
sudo yum install epel-release
sudo dnf install epel-release

# List repositories
yum repolist
dnf repolist

# History
yum history
dnf history

# Undo last transaction
sudo yum history undo last
sudo dnf history undo last

# Lock package version
sudo yum versionlock nginx
# Requires: yum install yum-plugin-versionlock

# Unlock package
sudo yum versionlock delete nginx

# Group install
sudo yum groupinstall "Development Tools"
sudo dnf groupinstall "Development Tools"

# List groups
yum grouplist
dnf grouplist
```

### RPM Commands (CentOS/RHEL)

```bash
# Install RPM package
sudo rpm -ivh package.rpm

# Upgrade RPM package
sudo rpm -Uvh package.rpm

# Remove RPM package
sudo rpm -e package

# Query installed package
rpm -q nginx

# List all installed packages
rpm -qa

# Show package info
rpm -qi nginx

# List files in package
rpm -ql nginx

# Show which package owns a file
rpm -qf /usr/sbin/nginx

# Verify package
rpm -V nginx

# Import GPG key
sudo rpm --import https://example.com/RPM-GPG-KEY
```

### DPKG Commands (Ubuntu/Debian)

```bash
# Install DEB package
sudo dpkg -i package.deb

# Remove package
sudo dpkg -r package

# Purge package
sudo dpkg -P package

# List installed packages
dpkg -l

# Show package info
dpkg -s nginx

# List files in package
dpkg -L nginx

# Show which package owns a file
dpkg -S /usr/sbin/nginx

# Reconfigure package
sudo dpkg-reconfigure nginx

# Fix broken dependencies
sudo dpkg --configure -a
sudo apt --fix-broken install
```

---

## System Information

```bash
# System information
uname -a                    # All system info
uname -r                    # Kernel version
uname -m                    # Machine architecture
hostname                    # System hostname
hostnamectl                 # Detailed hostname info

# OS information
cat /etc/os-release
lsb_release -a

# CPU information
lscpu
cat /proc/cpuinfo
nproc                       # Number of CPU cores

# Memory information
free -h
cat /proc/meminfo
vmstat
vmstat 1 5                  # Update every 1 second, 5 times

# Disk information
df -h                       # Disk space usage
df -i                       # Inode usage
lsblk                       # List block devices
fdisk -l                    # Partition information
blkid                       # Block device attributes

# Hardware information
lshw                        # List hardware
lshw -short
lspci                       # PCI devices
lsusb                       # USB devices
dmidecode                   # DMI/SMBIOS info
sudo dmidecode -t memory    # Memory info
sudo dmidecode -t processor # CPU info
sudo dmidecode -t bios      # BIOS info

# System uptime and load
uptime
w
who
last                        # Last logged in users
lastlog                     # Last login info

# Date and time
date
timedatectl
timedatectl set-timezone America/New_York
timedatectl list-timezones

# System logs
dmesg                       # Kernel ring buffer
dmesg | grep -i error
journalctl                  # Systemd logs
journalctl -u nginx         # Service logs
journalctl -f               # Follow logs
journalctl --since today
journalctl --since "2024-01-01" --until "2024-01-31"
```

---

## User & Group Management

### User Management

```bash
# Add user
sudo useradd john
sudo useradd -m john              # Create home directory
sudo useradd -m -s /bin/bash john # Specify shell

# Add user (interactive - Debian/Ubuntu)
sudo adduser john

# Set user password
sudo passwd john

# Modify user
sudo usermod -aG sudo john        # Add to sudo group (Ubuntu)
sudo usermod -aG wheel john       # Add to wheel group (CentOS)
sudo usermod -s /bin/zsh john     # Change shell
sudo usermod -d /home/newdir john # Change home directory
sudo usermod -l newname oldname   # Rename user

# Delete user
sudo userdel john
sudo userdel -r john              # Remove home directory

# Lock user account
sudo passwd -l john
sudo usermod -L john

# Unlock user account
sudo passwd -u john
sudo usermod -U john

# View user information
id john
finger john
getent passwd john
cat /etc/passwd | grep john

# List logged in users
who
w
users

# Switch user
su - john
sudo -i -u john

# Change password
passwd                            # Change own password
sudo passwd john                  # Change user password

# Set password expiry
sudo chage -E 2024-12-31 john    # Set expiry date
sudo chage -M 90 john            # Max password age
sudo chage -l john               # List password info

# Disable password expiry
sudo chage -M -1 john
```

### Group Management

```bash
# Create group
sudo groupadd developers

# Delete group
sudo groupdel developers

# Add user to group
sudo usermod -aG developers john
sudo gpasswd -a john developers

# Remove user from group
sudo gpasswd -d john developers

# List groups
cat /etc/group
getent group

# List user's groups
groups john
id john

# Change group password
sudo gpasswd developers

# Change primary group
sudo usermod -g developers john
```

### Sudo Configuration

```bash
# Edit sudoers file
sudo visudo

# Add user to sudoers
# Add line: john ALL=(ALL:ALL) ALL

# Allow user to run specific commands
# john ALL=(ALL) /usr/bin/systemctl restart nginx

# Allow group
# %developers ALL=(ALL:ALL) ALL

# No password sudo
# john ALL=(ALL) NOPASSWD: ALL

# Grant temporary sudo access
sudo usermod -aG sudo john        # Ubuntu
sudo usermod -aG wheel john       # CentOS

# Check sudo privileges
sudo -l
sudo -l -U john
```

---

## File & Directory Management

### Basic Operations

```bash
# List files
ls
ls -l                      # Long format
ls -a                      # Show hidden files
ls -lh                     # Human readable sizes
ls -ltr                    # Sort by time, reverse
ls -lS                     # Sort by size
ls -R                      # Recursive

# Create directory
mkdir mydir
mkdir -p path/to/mydir     # Create parent directories

# Remove directory
rmdir mydir                # Empty directory only
rm -r mydir                # Recursive removal
rm -rf mydir               # Force recursive removal

# Create file
touch file.txt
touch file1.txt file2.txt  # Multiple files

# Copy files
cp source.txt dest.txt
cp -r sourcedir destdir    # Recursive copy
cp -a sourcedir destdir    # Archive mode (preserve attributes)
cp -v source.txt dest.txt  # Verbose
cp -i source.txt dest.txt  # Interactive (confirm overwrite)

# Move/Rename
mv oldname.txt newname.txt
mv file.txt /path/to/destination/
mv -i file.txt dest.txt    # Interactive

# Remove files
rm file.txt
rm -i file.txt             # Interactive
rm -f file.txt             # Force
rm -rf directory/          # Force recursive

# View file content
cat file.txt
cat -n file.txt            # Show line numbers
tac file.txt               # Reverse order

# View file page by page
less file.txt
more file.txt

# View beginning of file
head file.txt
head -n 20 file.txt        # First 20 lines

# View end of file
tail file.txt
tail -n 20 file.txt        # Last 20 lines
tail -f file.txt           # Follow file updates (logs)
tail -f /var/log/syslog

# Create symbolic link
ln -s /path/to/file linkname

# Create hard link
ln /path/to/file linkname

# Find files
find /path -name "*.txt"
find /path -name "file*"
find /path -type f         # Files only
find /path -type d         # Directories only
find /path -mtime -7       # Modified in last 7 days
find /path -size +100M     # Larger than 100MB
find /path -user john      # Owned by john
find /path -perm 644       # Specific permissions

# Find and execute
find /path -name "*.log" -exec rm {} \;
find /path -name "*.txt" -exec chmod 644 {} \;

# Locate files (faster, uses database)
locate filename
sudo updatedb              # Update locate database

# Which command
which python
which -a python            # All instances

# File type
file filename

# Word count
wc file.txt
wc -l file.txt             # Line count
wc -w file.txt             # Word count
wc -c file.txt             # Byte count

# Disk usage
du -sh directory/          # Directory size
du -h --max-depth=1        # Subdirectory sizes
du -ah                     # All files and directories

# Compare files
diff file1.txt file2.txt
diff -u file1.txt file2.txt # Unified format
cmp file1.txt file2.txt    # Binary comparison

# File checksum
md5sum file.txt
sha256sum file.txt
sha512sum file.txt
```

---

## Permissions & Ownership

### Understanding Permissions

```
Format: drwxrwxrwx
        d = directory, - = file, l = link
        rwx = owner permissions (read, write, execute)
        rwx = group permissions
        rwx = other permissions

Numeric format:
r = 4, w = 2, x = 1
rwx = 7, rw- = 6, r-x = 5, r-- = 4
```

### Permission Commands

```bash
# Change file permissions
chmod 755 file.txt         # rwxr-xr-x
chmod 644 file.txt         # rw-r--r--
chmod 600 file.txt         # rw-------
chmod 777 file.txt         # rwxrwxrwx (avoid this!)

# Symbolic mode
chmod u+x file.txt         # Add execute for user
chmod g-w file.txt         # Remove write for group
chmod o+r file.txt         # Add read for others
chmod a+x file.txt         # Add execute for all
chmod u=rwx,g=rx,o=r file.txt

# Recursive
chmod -R 755 directory/

# Change ownership
sudo chown john file.txt
sudo chown john:developers file.txt
sudo chown -R john:developers directory/

# Change group only
sudo chgrp developers file.txt
sudo chgrp -R developers directory/

# Special permissions
chmod u+s file             # SUID - run as owner
chmod g+s directory        # SGID - inherit group
chmod +t directory         # Sticky bit - only owner can delete

# View permissions
ls -l file.txt
stat file.txt

# Default permissions (umask)
umask                      # View current umask
umask 022                  # Set umask
# umask 022 = files 644, directories 755
# umask 077 = files 600, directories 700

# ACL (Access Control Lists)
getfacl file.txt           # View ACL
setfacl -m u:john:rw file.txt  # Set ACL for user
setfacl -m g:developers:rx file.txt  # Set ACL for group
setfacl -x u:john file.txt # Remove ACL
setfacl -b file.txt        # Remove all ACLs

# Make file immutable
sudo chattr +i file.txt    # Cannot be deleted or modified
sudo chattr -i file.txt    # Remove immutable
lsattr file.txt            # View attributes
```

---

## Process Management

```bash
# List processes
ps
ps aux                     # All processes, detailed
ps -ef                     # All processes, full format
ps -u john                 # Processes by user
ps -C nginx                # Processes by name

# Process tree
pstree
pstree -p                  # Show PIDs

# Top (real-time monitoring)
top
htop                       # Enhanced version (install first)
# Top shortcuts:
# M - sort by memory
# P - sort by CPU
# k - kill process
# r - renice process

# Find process
pgrep nginx
pgrep -u john
pidof nginx

# Kill process
kill PID
kill -9 PID                # Force kill (SIGKILL)
kill -15 PID               # Graceful kill (SIGTERM)
kill -HUP PID              # Reload configuration (SIGHUP)

# Kill by name
pkill nginx
pkill -u john
killall nginx

# Background and foreground
command &                  # Run in background
jobs                       # List background jobs
fg %1                      # Bring job 1 to foreground
bg %1                      # Resume job 1 in background
nohup command &            # Run command immune to hangups

# Process priority
nice -n 10 command         # Start with priority
renice -n 5 -p PID         # Change priority
# Priority: -20 (highest) to 19 (lowest)

# Process limits
ulimit -a                  # Show all limits
ulimit -n 4096             # Set max open files
ulimit -u 2048             # Set max user processes

# Monitor process
watch -n 1 ps aux | grep nginx
watch -n 2 'df -h'         # Update every 2 seconds

# Process information
lsof                       # List open files
lsof -p PID                # Files opened by process
lsof -u john               # Files opened by user
lsof -i :80                # Processes using port 80
lsof -i TCP:22             # Processes on TCP port 22

# Resource usage
uptime
vmstat 1                   # Virtual memory stats
iostat                     # I/O statistics
sar -u 1 3                 # CPU usage (sysstat package)

# System calls
strace command             # Trace system calls
strace -p PID              # Trace running process

# Library calls
ltrace command
```

---

## Service Management

### Systemd (Modern Linux)

```bash
# Start service
sudo systemctl start nginx
sudo systemctl start mysql

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Enable service (start on boot)
sudo systemctl enable nginx

# Disable service
sudo systemctl disable nginx

# Enable and start
sudo systemctl enable --now nginx

# Check status
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx

# List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed

# View service logs
journalctl -u nginx
journalctl -u nginx -f          # Follow logs
journalctl -u nginx --since today
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx -n 50       # Last 50 lines

# Reload systemd
sudo systemctl daemon-reload

# List dependencies
systemctl list-dependencies nginx

# Show service properties
systemctl show nginx
systemctl cat nginx             # Show unit file

# Mask service (prevent start)
sudo systemctl mask nginx
sudo systemctl unmask nginx

# Edit service
sudo systemctl edit nginx       # Create override
sudo systemctl edit --full nginx  # Edit full unit file

# Analyze boot time
systemd-analyze
systemd-analyze blame           # Services by startup time
systemd-analyze critical-chain  # Critical path
```

### Service (SysV Init - Older Systems)

```bash
# Start service
sudo service nginx start

# Stop service
sudo service nginx stop

# Restart service
sudo service nginx restart

# Reload service
sudo service nginx reload

# Check status
sudo service nginx status

# List all services
service --status-all

# Enable service (Ubuntu/Debian)
sudo update-rc.d nginx enable

# Disable service (Ubuntu/Debian)
sudo update-rc.d nginx disable

# Enable service (CentOS/RHEL)
sudo chkconfig nginx on

# Disable service (CentOS/RHEL)
sudo chkconfig nginx off

# List services (CentOS/RHEL)
chkconfig --list
```

### Creating Systemd Service

```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start.sh
ExecStop=/opt/myapp/bin/stop.sh
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
# Reload and start
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

---

## Network Configuration

### Network Information

```bash
# IP address
ip addr show
ip a
ifconfig                   # Deprecated but still common

# Specific interface
ip addr show eth0
ifconfig eth0

# Routing table
ip route show
route -n
netstat -rn

# DNS configuration
cat /etc/resolv.conf

# Hostname
hostname
hostnamectl
sudo hostnamectl set-hostname newhostname

# Hosts file
cat /etc/hosts
sudo nano /etc/hosts

# Network interfaces
ls /sys/class/net/
ip link show

# Network statistics
netstat -s
ss -s

# Active connections
netstat -tunlp
ss -tunlp
ss -tulpn                  # All listening ports

# Specific port
netstat -tunlp | grep :80
ss -tulpn | grep :80
lsof -i :80

# Test connectivity
ping google.com
ping -c 4 google.com       # 4 packets only
ping6 google.com           # IPv6

# Trace route
traceroute google.com
tracepath google.com
mtr google.com             # Better traceroute

# DNS lookup
nslookup google.com
dig google.com
dig @8.8.8.8 google.com    # Specific DNS server
host google.com

# Port scanning
nc -zv google.com 80       # Check if port is open
nmap localhost             # Scan local ports (install nmap)
nmap -p 80,443 example.com # Scan specific ports

# Download files
wget https://example.com/file.zip
wget -O custom-name.zip https://example.com/file.zip
curl -O https://example.com/file.zip
curl -L https://example.com/file.zip  # Follow redirects

# Test HTTP
curl https://example.com
curl -I https://example.com   # Headers only
curl -v https://example.com   # Verbose

# Network speed test
speedtest-cli              # Install first
```

### Ubuntu/Debian Network Configuration

#### Netplan (Ubuntu 18.04+)

```bash
# Edit netplan config
sudo nano /etc/netplan/01-netcfg.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

```bash
# Apply configuration
sudo netplan apply

# Test configuration
sudo netplan try

# Debug
sudo netplan --debug apply
```

#### Old Method (/etc/network/interfaces)

```bash
sudo nano /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

```bash
# Restart networking
sudo systemctl restart networking
sudo ifdown eth0 && sudo ifup eth0
```

### CentOS/RHEL Network Configuration

```bash
# Edit network interface
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
```

```
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

```bash
# Restart network
sudo systemctl restart network
sudo nmcli connection reload
sudo nmcli connection up eth0

# Using nmcli (NetworkManager)
nmcli device status
nmcli connection show
nmcli connection modify eth0 ipv4.addresses 192.168.1.100/24
nmcli connection modify eth0 ipv4.gateway 192.168.1.1
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection modify eth0 ipv4.method manual
nmcli connection up eth0
```

### Network Tools

```bash
# Enable/disable interface
sudo ip link set eth0 up
sudo ip link set eth0 down
sudo ifconfig eth0 up
sudo ifconfig eth0 down

# Add IP address
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Add route
sudo ip route add 192.168.2.0/24 via 192.168.1.1
sudo route add -net 192.168.2.0/24 gw 192.168.1.1

# Delete route
sudo ip route del 192.168.2.0/24
sudo route del -net 192.168.2.0/24

# Set default gateway
sudo ip route add default via 192.168.1.1
sudo route add default gw 192.168.1.1

# ARP table
ip neigh show
arp -a

# Flush ARP cache
sudo ip neigh flush all

# Network bandwidth
iftop                      # Install first
iftop -i eth0
nethogs                    # Per-process bandwidth
vnstat                     # Network traffic monitor
```

---

## Disk & Storage Management

### Disk Information

```bash
# Disk usage
df -h                      # Human readable
df -i                      # Inode usage
df -T                      # Show filesystem type

# Directory size
du -sh /path/to/dir
du -h --max-depth=1 /path
du -ah /path | sort -hr | head -20  # Top 20 largest

# List block devices
lsblk
lsblk -f                   # Show filesystem
fdisk -l

# Disk statistics
iostat
iostat -x 1                # Extended stats, 1 sec interval

# Partition information
sudo parted -l
sudo fdisk -l
```

### Partitioning

```bash
# Using fdisk
sudo fdisk /dev/sdb
# Commands within fdisk:
# n - new partition
# d - delete partition
# p - print partition table
# w - write changes
# q - quit without saving

# Using parted
sudo parted /dev/sdb
(parted) print
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) quit

# Using gdisk (GPT)
sudo gdisk /dev/sdb
```

### Filesystem Operations

```bash
# Create filesystem
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb1
sudo mkfs.btrfs /dev/sdb1

# Create filesystem with label
sudo mkfs.ext4 -L mydata /dev/sdb1

# Check filesystem
sudo fsck /dev/sdb1
sudo fsck.ext4 /dev/sdb1
sudo xfs_repair /dev/sdb1

# Mount filesystem
sudo mount /dev/sdb1 /mnt/data

# Mount with options
sudo mount -t ext4 -o rw,noatime /dev/sdb1 /mnt/data

# Unmount
sudo umount /mnt/data
sudo umount /dev/sdb1

# Force unmount
sudo umount -f /mnt/data
sudo umount -l /mnt/data   # Lazy unmount

# View mounts
mount
mount | grep sdb1
cat /proc/mounts
findmnt

# /etc/fstab (persistent mounts)
sudo nano /etc/fstab
```

```
# <device>    <mount point>  <type>  <options>        <dump> <pass>
/dev/sdb1     /mnt/data      ext4    defaults         0      2
UUID=xxx      /mnt/data      ext4    defaults,noatime 0      2
```

```bash
# Get UUID
sudo blkid /dev/sdb1
lsblk -f

# Mount all from fstab
sudo mount -a

# Remount
sudo mount -o remount,rw /mnt/data
```

### LVM (Logical Volume Manager)

```bash
# Install LVM
sudo apt install lvm2        # Ubuntu/Debian
sudo yum install lvm2        # CentOS/RHEL

# Create physical volume
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc

# Display physical volumes
sudo pvdisplay
sudo pvs

# Create volume group
sudo vgcreate vg_data /dev/sdb /dev/sdc

# Display volume groups
sudo vgdisplay
sudo vgs

# Extend volume group
sudo vgextend vg_data /dev/sdd

# Create logical volume
sudo lvcreate -L 100G -n lv_data vg_data
sudo lvcreate -l 100%FREE -n lv_data vg_data  # Use all space

# Display logical volumes
sudo lvdisplay
sudo lvs

# Create filesystem on LV
sudo mkfs.ext4 /dev/vg_data/lv_data

# Mount LV
sudo mount /dev/vg_data/lv_data /mnt/data

# Extend logical volume
sudo lvextend -L +50G /dev/vg_data/lv_data
sudo lvextend -l +100%FREE /dev/vg_data/lv_data

# Resize filesystem (ext4)
sudo resize2fs /dev/vg_data/lv_data

# Resize filesystem (xfs)
sudo xfs_growfs /mnt/data

# Reduce logical volume (ext4 only, unmount first!)
sudo umount /mnt/data
sudo e2fsck -f /dev/vg_data/lv_data
sudo resize2fs /dev/vg_data/lv_data 50G
sudo lvreduce -L 50G /dev/vg_data/lv_data

# Remove logical volume
sudo umount /mnt/data
sudo lvremove /dev/vg_data/lv_data

# Remove volume group
sudo vgremove vg_data

# Remove physical volume
sudo pvremove /dev/sdb
```

### RAID Management

```bash
# Install mdadm
sudo apt install mdadm       # Ubuntu/Debian
sudo yum install mdadm       # CentOS/RHEL

# Create RAID 1 (mirror)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Create RAID 5
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# Create RAID 10
sudo mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sd{b,c,d,e}

# Check RAID status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Add spare device
sudo mdadm --add /dev/md0 /dev/sdf

# Remove failed device
sudo mdadm --fail /dev/md0 /dev/sdc
sudo mdadm --remove /dev/md0 /dev/sdc

# Save configuration
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

# Stop RAID
sudo mdadm --stop /dev/md0

# Remove RAID
sudo mdadm --zero-superblock /dev/sdb /dev/sdc
```

### NFS (Network File System)

```bash
# Install NFS server
sudo apt install nfs-kernel-server  # Ubuntu/Debian
sudo yum install nfs-utils          # CentOS/RHEL

# Configure exports
sudo nano /etc/exports
```

```
/mnt/data 192.168.1.0/24(rw,sync,no_subtree_check)
/mnt/share *(ro,sync,no_subtree_check)
```

```bash
# Export shares
sudo exportfs -a
sudo exportfs -ra              # Re-export all

# Show exports
sudo exportfs -v

# Restart NFS server
sudo systemctl restart nfs-server       # Ubuntu
sudo systemctl restart nfs-kernel-server # Debian
sudo systemctl restart nfs              # CentOS

# NFS client - install
sudo apt install nfs-common    # Ubuntu/Debian
sudo yum install nfs-utils     # CentOS/RHEL

# Mount NFS share
sudo mount -t nfs 192.168.1.10:/mnt/data /mnt/remote

# Permanent mount in /etc/fstab
192.168.1.10:/mnt/data  /mnt/remote  nfs  defaults  0  0

# Show NFS mounts
showmount -e 192.168.1.10
```

---

## SSH & Remote Access

### SSH Configuration

```bash
# Install SSH server
sudo apt install openssh-server  # Ubuntu/Debian
sudo yum install openssh-server  # CentOS/RHEL

# Start SSH service
sudo systemctl start sshd
sudo systemctl enable sshd

# SSH configuration file
sudo nano /etc/ssh/sshd_config
```

```
Port 22
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
AllowUsers user1 user2
```

```bash
# Restart SSH after config changes
sudo systemctl restart sshd

# SSH client
ssh user@hostname
ssh user@192.168.1.100
ssh -p 2222 user@hostname     # Non-standard port

# SSH with key
ssh -i ~/.ssh/id_rsa user@hostname

# Generate SSH key pair
ssh-keygen
ssh-keygen -t rsa -b 4096 -C "email@example.com"
ssh-keygen -t ed25519

# Copy public key to server
ssh-copy-id user@hostname
ssh-copy-id -i ~/.ssh/id_rsa.pub user@hostname

# Manual key copy
cat ~/.ssh/id_rsa.pub | ssh user@hostname 'cat >> ~/.ssh/authorized_keys'

# SSH config file
nano ~/.ssh/config
```

```
Host myserver
    HostName 192.168.1.100
    User john
    Port 22
    IdentityFile ~/.ssh/id_rsa

Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

```bash
# Use config
ssh myserver

# SSH tunneling (port forwarding)
ssh -L 8080:localhost:80 user@hostname  # Local forward
ssh -R 8080:localhost:80 user@hostname  # Remote forward
ssh -D 1080 user@hostname               # Dynamic (SOCKS proxy)

# SSH with X11 forwarding
ssh -X user@hostname
ssh -Y user@hostname          # Trusted X11 forwarding

# Keep SSH connection alive
ssh -o ServerAliveInterval=60 user@hostname

# SCP (Secure Copy)
scp file.txt user@hostname:/path/
scp user@hostname:/path/file.txt ./
scp -r directory/ user@hostname:/path/  # Recursive
scp -P 2222 file.txt user@hostname:/path/  # Non-standard port

# SFTP (SSH File Transfer Protocol)
sftp user@hostname
# Commands: ls, cd, get, put, mget, mput, exit

# Rsync over SSH
rsync -avz -e ssh /local/path/ user@hostname:/remote/path/
rsync -avz --progress -e ssh /local/ user@host:/remote/
rsync -avz --delete -e ssh /local/ user@host:/remote/  # Delete extra files

# SSH agent
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa
ssh-add -l                    # List added keys

# Disable host key checking (not recommended)
ssh -o StrictHostKeyChecking=no user@hostname

# SSH escape sequences
# ~. - terminate connection
# ~^Z - suspend ssh
# ~# - list forwarded connections
```

### VNC Setup

```bash
# Install VNC server (Ubuntu/Debian)
sudo apt install tightvncserver

# Install VNC server (CentOS/RHEL)
sudo yum install tigervnc-server

# Start VNC server
vncserver :1
vncserver -geometry 1920x1080 :1

# Set VNC password
vncpasswd

# Kill VNC server
vncserver -kill :1

# VNC client
vncviewer hostname:1
```

---

## Firewall Configuration

### UFW (Ubuntu/Debian)

```bash
# Install UFW
sudo apt install ufw

# Enable/Disable
sudo ufw enable
sudo ufw disable

# Status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow port
sudo ufw allow 22
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow port from specific IP
sudo ufw allow from 192.168.1.100 to any port 22

# Allow port range
sudo ufw allow 6000:6007/tcp

# Allow service
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Deny port
sudo ufw deny 23

# Delete rule
sudo ufw delete allow 80
sudo ufw delete 2              # By rule number

# Application profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'
sudo ufw allow 'Apache Full'

# Logging
sudo ufw logging on
sudo ufw logging medium

# Reset firewall
sudo ufw reset
```

### Firewalld (CentOS/RHEL)

```bash
# Install firewalld
sudo yum install firewalld

# Start/Enable
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Status
sudo firewall-cmd --state
sudo firewall-cmd --list-all
sudo firewall-cmd --list-all-zones

# Get default zone
sudo firewall-cmd --get-default-zone

# Set default zone
sudo firewall-cmd --set-default-zone=public

# List services
sudo firewall-cmd --get-services
sudo firewall-cmd --list-services

# Add service
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --add-service=ssh

# Remove service
sudo firewall-cmd --remove-service=http

# Add port
sudo firewall-cmd --add-port=8080/tcp
sudo firewall-cmd --add-port=8000-9000/tcp

# Remove port
sudo firewall-cmd --remove-port=8080/tcp

# Make permanent
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

# Add rich rule
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.100" accept'

# Forward port
sudo firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080

# List ports
sudo firewall-cmd --list-ports

# Zone management
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --zone=public --add-service=http

# Panic mode (block all)
sudo firewall-cmd --panic-on
sudo firewall-cmd --panic-off
```

### Iptables

```bash
# List rules
sudo iptables -L
sudo iptables -L -v
sudo iptables -L -n --line-numbers

# List NAT rules
sudo iptables -t nat -L

# Allow port
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Drop traffic
sudo iptables -A INPUT -s 192.168.1.200 -j DROP

# Delete rule
sudo iptables -D INPUT 3     # Delete rule number 3

# Flush rules
sudo iptables -F

# Default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Save rules (Ubuntu/Debian)
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4

# Save rules (CentOS/RHEL)
sudo service iptables save
sudo iptables-save > /etc/sysconfig/iptables

# NAT (masquerading)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

---

## Performance Monitoring

### CPU Monitoring

```bash
# Top
top
htop
atop

# CPU info
lscpu
cat /proc/cpuinfo
mpstat                     # CPU statistics
mpstat 1 5                 # 1 sec interval, 5 times
mpstat -P ALL              # Per-CPU stats

# Load average
uptime
cat /proc/loadavg
w

# CPU usage by process
ps aux --sort=-%cpu | head -10
```

### Memory Monitoring

```bash
# Memory usage
free -h
free -m
cat /proc/meminfo

# Memory by process
ps aux --sort=-%mem | head -10

# Detailed memory
vmstat 1
vmstat 1 5

# Swap usage
swapon --show
cat /proc/swaps

# Clear cache (don't do this in production!)
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

### Disk I/O Monitoring

```bash
# Disk I/O statistics
iostat
iostat -x 1                # Extended stats, 1 sec
iostat -d 2 6              # Device stats, 2 sec, 6 times

# Disk usage by directory
du -sh /*
du -h --max-depth=1 / | sort -hr

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
find / -type f -size +1G 2>/dev/null

# I/O by process
iotop                      # Install first
iotop -o                   # Only show active

# Block device stats
cat /proc/diskstats
```

### Network Monitoring

```bash
# Network statistics
netstat -s
ss -s

# Bandwidth usage
iftop -i eth0              # Install first
nload eth0
bmon

# Connection tracking
netstat -an | grep ESTABLISHED | wc -l
ss -s

# TCP connection states
netstat -ant | awk '{print $6}' | sort | uniq -c | sort -n

# Network throughput
iperf -s                   # Server
iperf -c server_ip         # Client

# Packet capture
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 -w capture.pcap
sudo tcpdump -i eth0 -n host 192.168.1.100
```

### System Monitoring

```bash
# All-in-one monitoring
htop
glances                    # Install first
nmon

# System statistics
sar -u 1 3                 # CPU (sysstat package)
sar -r 1 3                 # Memory
sar -n DEV 1 3             # Network
sar -d 1 3                 # Disk

# System activity report
sar -A

# Process accounting
sa
lastcomm

# Detailed performance
perf top
perf stat command

# Trace system calls
strace -c command          # Count calls
strace -p PID              # Trace running process
```

---

## Log Management

### System Logs Location

```bash
# Ubuntu/Debian
/var/log/syslog            # System log
/var/log/auth.log          # Authentication log
/var/log/kern.log          # Kernel log
/var/log/dpkg.log          # Package management
/var/log/apt/              # APT logs

# CentOS/RHEL
/var/log/messages          # System log
/var/log/secure            # Authentication log
/var/log/maillog           # Mail server log
/var/log/yum.log           # YUM package management

# Common logs
/var/log/boot.log          # Boot messages
/var/log/dmesg             # Kernel ring buffer
/var/log/cron              # Cron jobs
/var/log/nginx/            # Nginx logs
/var/log/apache2/          # Apache logs
/var/log/mysql/            # MySQL logs
```

### Viewing Logs

```bash
# View system log
sudo tail -f /var/log/syslog       # Ubuntu
sudo tail -f /var/log/messages     # CentOS

# Authentication logs
sudo tail -f /var/log/auth.log     # Ubuntu
sudo tail -f /var/log/secure       # CentOS

# Last 100 lines
sudo tail -n 100 /var/log/syslog

# Kernel messages
dmesg
dmesg | tail
dmesg | grep -i error
dmesg -T                   # Human readable timestamps
dmesg -w                   # Follow mode

# Search logs
grep "error" /var/log/syslog
grep -i "failed" /var/log/auth.log
grep -r "error" /var/log/

# Filter by time
grep "Dec  6 14:" /var/log/syslog
```

### Journalctl (Systemd)

```bash
# View all logs
journalctl

# Follow logs
journalctl -f

# Show last N lines
journalctl -n 50
journalctl -n 100 -f

# Logs for specific service
journalctl -u nginx
journalctl -u sshd -f

# Logs since boot
journalctl -b
journalctl -b -1           # Previous boot

# Time range
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl --since "10 min ago"
journalctl --since today
journalctl --since yesterday
journalctl --since "2024-01-01" --until "2024-01-31"

# Priority level
journalctl -p err          # Errors only
journalctl -p warning      # Warnings and above

# Kernel messages
journalctl -k
journalctl -k -f

# By user
journalctl _UID=1000

# Disk usage
journalctl --disk-usage

# Vacuum old logs
sudo journalctl --vacuum-time=2weeks
sudo journalctl --vacuum-size=1G

# Verify logs
sudo journalctl --verify

# Export logs
journalctl -o json         # JSON format
journalctl -o json-pretty
journalctl > /tmp/logs.txt
```

### Logrotate

```bash
# Logrotate configuration
sudo nano /etc/logrotate.conf
sudo nano /etc/logrotate.d/nginx
```

```
/var/log/nginx/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}
```

```bash
# Test logrotate
sudo logrotate -d /etc/logrotate.d/nginx  # Debug mode
sudo logrotate -f /etc/logrotate.d/nginx  # Force rotation

# Manual run
sudo logrotate /etc/logrotate.conf
```

### Rsyslog

```bash
# Rsyslog configuration
sudo nano /etc/rsyslog.conf
sudo nano /etc/rsyslog.d/50-default.conf

# Restart rsyslog
sudo systemctl restart rsyslog

# Send logs to remote server
# Add to /etc/rsyslog.conf:
*.* @192.168.1.100:514     # UDP
*.* @@192.168.1.100:514    # TCP

# Test logging
logger "Test message"
logger -p user.err "Error message"
```

---

## Cron Jobs & Scheduling

### Crontab

```bash
# Edit crontab
crontab -e                 # Current user
sudo crontab -e            # Root
sudo crontab -e -u john    # Specific user

# List crontab
crontab -l
sudo crontab -l -u john

# Remove crontab
crontab -r

# Crontab format
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └─── Day of week (0-7, Sun=0 or 7)
# │ │ │ └───── Month (1-12)
# │ │ └─────── Day of month (1-31)
# │ └───────── Hour (0-23)
# └─────────── Minute (0-59)

# Examples:
# Run every minute
* * * * * /path/to/script.sh

# Run every hour at minute 0
0 * * * * /path/to/script.sh

# Run every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Run every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run every 15 minutes
*/15 * * * * /path/to/script.sh

# Run on specific days (Mon, Wed, Fri)
0 9 * * 1,3,5 /path/to/script.sh

# Run on weekdays (Mon-Fri)
0 9 * * 1-5 /path/to/script.sh

# Run first day of month
0 0 1 * * /path/to/script.sh

# Run at reboot
@reboot /path/to/script.sh

# Special strings
@yearly    # 0 0 1 1 *
@annually  # 0 0 1 1 *
@monthly   # 0 0 1 * *
@weekly    # 0 0 * * 0
@daily     # 0 0 * * *
@midnight  # 0 0 * * *
@hourly    # 0 * * * *

# With environment variables
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=admin@example.com

0 2 * * * /path/to/backup.sh

# Redirect output
0 2 * * * /path/to/script.sh > /var/log/script.log 2>&1
0 2 * * * /path/to/script.sh >> /var/log/script.log 2>&1  # Append

# System cron directories
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

### At Command

```bash
# Schedule one-time job
echo "/path/to/script.sh" | at 10:00
at 10:00 -f /path/to/script.sh
at now + 1 hour
at 2:30 PM tomorrow
at 10:00 AM 12/25/2024

# Interactive mode
at 10:00
> /path/to/command
> Ctrl+D

# List scheduled jobs
atq
at -l

# Remove job
atrm job_number
at -d job_number

# Show job details
at -c job_number
```

### Systemd Timers

```bash
# Create timer unit
sudo nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Create service unit
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Backup service

[Service]
Type=oneshot
ExecStart=/path/to/backup.sh
```

```bash
# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Check timer status
systemctl list-timers
systemctl status backup.timer
```

---

## Shell Scripting

### Basic Script Structure

```bash
#!/bin/bash

# Simple script
echo "Hello World"

# Variables
NAME="John"
echo "Hello, $NAME"

# Command substitution
DATE=$(date +%Y-%m-%d)
USERS=`who | wc -l`

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"

# If statement
if [ "$NAME" == "John" ]; then
    echo "Hello John"
elif [ "$NAME" == "Jane" ]; then
    echo "Hello Jane"
else
    echo "Hello stranger"
fi

# Comparison operators
# -eq  Equal
# -ne  Not equal
# -gt  Greater than
# -ge  Greater than or equal
# -lt  Less than
# -le  Less than or equal

# File tests
# -f   File exists
# -d   Directory exists
# -r   Readable
# -w   Writable
# -x   Executable
# -s   File size > 0

if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing $file"
done

for ((i=1; i<=10; i++)); do
    echo "Count: $i"
done

# While loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Until loop
COUNT=1
until [ $COUNT -gt 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Functions
function greet() {
    echo "Hello, $1"
}

greet "John"

# Return value
function add() {
    return $(($1 + $2))
}

add 5 3
echo "Result: $?"

# Arrays
FRUITS=("Apple" "Banana" "Orange")
echo ${FRUITS[0]}
echo ${FRUITS[@]}      # All elements
echo ${#FRUITS[@]}     # Array length

# Case statement
case $1 in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac

# Exit codes
exit 0  # Success
exit 1  # Error

# Special variables
$0      # Script name
$1-$9   # Arguments
$#      # Number of arguments
$@      # All arguments
$?      # Last exit code
$$      # Process ID
$!      # Last background process ID
```

### Script Examples

#### Backup Script

```bash
#!/bin/bash

# Backup script
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

# Create backup
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"

# Check if successful
if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
else
    echo "Backup failed!"
    exit 1
fi

# Delete backups older than 7 days
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete

echo "Backup completed"
```

#### System Information Script

```bash
#!/bin/bash

echo "=== System Information ==="
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo ""
echo "=== CPU ==="
echo "CPU Model: $(lscpu | grep 'Model name' | cut -d':' -f2 | xargs)"
echo "CPU Cores: $(nproc)"
echo ""
echo "=== Memory ==="
free -h
echo ""
echo "=== Disk ==="
df -h | grep -v tmpfs
echo ""
echo "=== Network ==="
ip -4 addr show | grep inet
```

#### Service Monitor Script

```bash
#!/bin/bash

SERVICE="nginx"

if systemctl is-active --quiet $SERVICE; then
    echo "$SERVICE is running"
else
    echo "$SERVICE is not running. Starting..."
    systemctl start $SERVICE

    if systemctl is-active --quiet $SERVICE; then
        echo "$SERVICE started successfully"
    else
        echo "Failed to start $SERVICE"
        exit 1
    fi
fi
```

---

## Text Processing

### grep

```bash
# Basic search
grep "pattern" file.txt
grep "error" /var/log/syslog

# Case insensitive
grep -i "error" file.txt

# Recursive search
grep -r "pattern" /path/to/directory
grep -R "pattern" .

# Show line numbers
grep -n "pattern" file.txt

# Invert match
grep -v "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only matching part
grep -o "pattern" file.txt

# Extended regex
grep -E "pattern1|pattern2" file.txt
egrep "pattern1|pattern2" file.txt

# Show context
grep -A 3 "pattern" file.txt  # 3 lines after
grep -B 3 "pattern" file.txt  # 3 lines before
grep -C 3 "pattern" file.txt  # 3 lines before and after

# Multiple patterns
grep -e "pattern1" -e "pattern2" file.txt

# From multiple files
grep "pattern" file1.txt file2.txt

# Show filenames only
grep -l "pattern" *.txt

# Search compressed files
zgrep "pattern" file.gz
```

### sed

```bash
# Replace first occurrence
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace in-place
sed -i 's/old/new/g' file.txt

# Replace with backup
sed -i.bak 's/old/new/g' file.txt

# Delete lines
sed '/pattern/d' file.txt
sed '5d' file.txt              # Delete line 5
sed '5,10d' file.txt           # Delete lines 5-10

# Print specific lines
sed -n '5p' file.txt           # Print line 5
sed -n '5,10p' file.txt        # Print lines 5-10
sed -n '/pattern/p' file.txt   # Print matching lines

# Multiple commands
sed -e 's/old1/new1/g' -e 's/old2/new2/g' file.txt

# Add line after match
sed '/pattern/a\New line' file.txt

# Add line before match
sed '/pattern/i\New line' file.txt

# Change line
sed '/pattern/c\New line' file.txt
```

### awk

```bash
# Print columns
awk '{print $1}' file.txt      # First column
awk '{print $1,$3}' file.txt   # First and third columns
awk '{print $NF}' file.txt     # Last column

# With delimiter
awk -F: '{print $1}' /etc/passwd
awk -F, '{print $1,$2}' file.csv

# Pattern matching
awk '/pattern/ {print $0}' file.txt
awk '$3 > 100 {print $1,$3}' file.txt

# Built-in variables
awk '{print NR, $0}' file.txt  # Line number and line
awk '{print NF}' file.txt      # Number of fields

# Sum column
awk '{sum += $1} END {print sum}' file.txt

# Average
awk '{sum += $1; count++} END {print sum/count}' file.txt

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Format output
awk '{printf "%-10s %s\n", $1, $2}' file.txt
```

### cut

```bash
# Cut by character position
cut -c1-10 file.txt            # Characters 1-10

# Cut by field
cut -f1 file.txt               # First field
cut -f1,3 file.txt             # First and third fields
cut -f1-3 file.txt             # Fields 1-3

# Custom delimiter
cut -d: -f1 /etc/passwd        # First field, : delimiter
cut -d, -f1,2 file.csv
```

### sort

```bash
# Sort alphabetically
sort file.txt

# Sort numerically
sort -n file.txt

# Reverse sort
sort -r file.txt

# Sort by column
sort -k2 file.txt              # Sort by second column
sort -t: -k3 -n /etc/passwd    # Sort by third field, numeric

# Unique
sort -u file.txt
```

### uniq

```bash
# Remove duplicates (requires sorted input)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Show only duplicates
sort file.txt | uniq -d

# Show only unique lines
sort file.txt | uniq -u
```

### tr

```bash
# Replace characters
echo "hello" | tr 'a-z' 'A-Z'  # Uppercase
echo "HELLO" | tr 'A-Z' 'a-z'  # Lowercase

# Delete characters
echo "hello123" | tr -d '0-9'  # Remove numbers

# Squeeze repeats
echo "hello    world" | tr -s ' '
```

---

## Archive & Compression

### tar

```bash
# Create archive
tar -cf archive.tar file1 file2
tar -czf archive.tar.gz directory/      # Gzip compression
tar -cjf archive.tar.bz2 directory/     # Bzip2 compression
tar -cJf archive.tar.xz directory/      # XZ compression

# Create with verbose
tar -cvzf archive.tar.gz directory/

# Extract archive
tar -xf archive.tar
tar -xzf archive.tar.gz
tar -xjf archive.tar.bz2
tar -xJf archive.tar.xz

# Extract to specific directory
tar -xzf archive.tar.gz -C /path/to/dir

# List archive contents
tar -tzf archive.tar.gz
tar -tf archive.tar

# Exclude files
tar -czf archive.tar.gz --exclude='*.log' directory/

# Append to archive
tar -rf archive.tar newfile.txt

# Extract specific file
tar -xzf archive.tar.gz file.txt

# Preserve permissions
tar -czpf archive.tar.gz directory/
```

### gzip/gunzip

```bash
# Compress file
gzip file.txt                  # Creates file.txt.gz
gzip -k file.txt               # Keep original

# Decompress
gunzip file.txt.gz
gzip -d file.txt.gz

# Compression level (1-9)
gzip -9 file.txt               # Best compression

# View compressed file
zcat file.txt.gz
zless file.txt.gz
zgrep "pattern" file.txt.gz
```

### bzip2/bunzip2

```bash
# Compress
bzip2 file.txt
bzip2 -k file.txt              # Keep original

# Decompress
bunzip2 file.txt.bz2
bzip2 -d file.txt.bz2

# View compressed file
bzcat file.txt.bz2
bzless file.txt.bz2
bzgrep "pattern" file.txt.bz2
```

### zip/unzip

```bash
# Create zip
zip archive.zip file1 file2
zip -r archive.zip directory/

# Add to existing zip
zip archive.zip newfile.txt

# Decompress
unzip archive.zip
unzip archive.zip -d /path/to/dir

# List contents
unzip -l archive.zip

# Test zip file
unzip -t archive.zip

# Exclude files
zip -r archive.zip directory/ -x "*.log"

# Password protect
zip -e -r archive.zip directory/
unzip -P password archive.zip
```

---

## Security Hardening

### Update System

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y

# CentOS/RHEL
sudo yum update -y
sudo dnf update -y
```

### Secure SSH

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

```
# Disable root login
PermitRootLogin no

# Use key-based authentication only
PasswordAuthentication no
PubkeyAuthentication yes

# Change default port
Port 2222

# Limit users
AllowUsers user1 user2

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding
X11Forwarding no

# Use protocol 2 only
Protocol 2

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
# Restart SSH
sudo systemctl restart sshd
```

### Fail2ban

```bash
# Install
sudo apt install fail2ban       # Ubuntu/Debian
sudo yum install fail2ban        # CentOS/RHEL

# Configure
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
```

```bash
# Start fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### Disable Unnecessary Services

```bash
# List running services
systemctl list-units --type=service --state=running

# Disable service
sudo systemctl disable service_name
sudo systemctl stop service_name
```

### File Integrity Monitoring

```bash
# Install AIDE
sudo apt install aide            # Ubuntu/Debian
sudo yum install aide            # CentOS/RHEL

# Initialize database
sudo aideinit

# Check integrity
sudo aide --check
```

### Security Auditing

```bash
# Install Lynis
sudo apt install lynis           # Ubuntu/Debian
sudo yum install lynis           # CentOS/RHEL

# Run audit
sudo lynis audit system

# Install ClamAV
sudo apt install clamav clamav-daemon
sudo freshclam                   # Update virus definitions
sudo clamscan -r /home           # Scan directory
```

---

## Troubleshooting

### Common Issues

```bash
# Out of disk space
df -h
du -sh /*
du -h --max-depth=1 / | sort -hr

# High CPU usage
top
ps aux --sort=-%cpu | head -10

# High memory usage
free -h
ps aux --sort=-%mem | head -10

# Network issues
ping google.com
ip addr show
ip route show
cat /etc/resolv.conf
systemctl status NetworkManager

# DNS issues
nslookup google.com
dig google.com
cat /etc/resolv.conf

# Service not starting
systemctl status service_name
journalctl -u service_name -n 50
journalctl -u service_name --since "1 hour ago"

# Permission denied
ls -l filename
sudo chmod permissions filename
sudo chown user:group filename

# Command not found
which command_name
echo $PATH
sudo apt install package_name

# Port already in use
sudo lsof -i :80
sudo netstat -tulpn | grep :80
sudo kill -9 PID
```

### System Recovery

```bash
# Boot into single user mode
# Add to kernel parameters: single or 1

# Boot into rescue mode
systemctl rescue

# Reset root password (Ubuntu)
# 1. Boot into recovery mode
# 2. Drop to root shell
mount -o remount,rw /
passwd root
reboot

# Check filesystem
sudo fsck /dev/sda1
sudo fsck -y /dev/sda1         # Auto-repair

# Repair boot
sudo update-grub               # Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # CentOS
```

---

## Advanced Topics

### Kernel Management

```bash
# View kernel version
uname -r

# List installed kernels
dpkg --list | grep linux-image  # Ubuntu/Debian
rpm -qa | grep kernel           # CentOS/RHEL

# Remove old kernels (Ubuntu)
sudo apt autoremove --purge

# Update initramfs
sudo update-initramfs -u        # Ubuntu/Debian
sudo dracut -f                  # CentOS/RHEL

# Kernel parameters
cat /proc/cmdline
sysctl -a
sudo sysctl -w parameter=value

# Make persistent
sudo nano /etc/sysctl.conf
sudo sysctl -p
```

### SELinux (CentOS/RHEL)

```bash
# Check status
sestatus
getenforce

# Set mode
sudo setenforce 0              # Permissive
sudo setenforce 1              # Enforcing

# Permanent change
sudo nano /etc/selinux/config
# SELINUX=enforcing|permissive|disabled

# View contexts
ls -Z
ps -Z

# Change context
sudo chcon -t httpd_sys_content_t /var/www/html/file.txt

# Restore default context
sudo restorecon -R /var/www/html

# SELinux booleans
getsebool -a
sudo setsebool -P httpd_can_network_connect on

# Troubleshoot
sudo ausearch -m avc -ts recent
sudo audit2why < /var/log/audit/audit.log
sudo audit2allow -M mypolicy < /var/log/audit/audit.log
sudo semodule -i mypolicy.pp
```

### AppArmor (Ubuntu)

```bash
# Check status
sudo apparmor_status

# Profiles location
ls /etc/apparmor.d/

# Set to complain mode
sudo aa-complain /etc/apparmor.d/usr.bin.firefox

# Set to enforce mode
sudo aa-enforce /etc/apparmor.d/usr.bin.firefox

# Disable profile
sudo ln -s /etc/apparmor.d/usr.bin.firefox /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/usr.bin.firefox

# Enable profile
sudo rm /etc/apparmor.d/disable/usr.bin.firefox
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.firefox
```

---

## Conclusion

This comprehensive Linux guide covers essential commands and configurations for Ubuntu, CentOS/RHEL, and Debian distributions. Key takeaways:

1. **Package Management**: Use APT for Debian-based, YUM/DNF for RHEL-based
2. **Service Management**: Systemd is standard across modern distributions
3. **Security**: Regular updates, firewall, SSH hardening, fail2ban
4. **Monitoring**: Use top/htop, logs, and journalctl
5. **Networking**: Understand IP configuration, firewall rules, SSH
6. **Storage**: LVM for flexibility, proper filesystem management
7. **Automation**: Cron jobs and systemd timers for scheduled tasks

### Additional Resources

- [Ubuntu Documentation](https://help.ubuntu.com/)
- [CentOS Documentation](https://docs.centos.org/)
- [Debian Administrator's Handbook](https://www.debian.org/doc/)
- [Linux Documentation Project](https://tldp.org/)
- [Arch Wiki](https://wiki.archlinux.org/) - Excellent resource for all distros

---

**Remember**: Always test commands in a non-production environment first, maintain regular backups, and keep your system updated!
