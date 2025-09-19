# cis-harden
CIS Hardening

---

# 🔐 CIS Benchmark Ubuntu 22.04 LTS v1.0.0 – Configuration Guide

---

## **1. Filesystem Configuration**

### 1.1.1 Disable unused filesystems

แก้ไขไฟล์ `/etc/modprobe.d/CIS.conf`

```bash
echo "install cramfs /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install freevxfs /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install jffs2 /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install hfs /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install hfsplus /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install squashfs /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install udf /bin/true" >> /etc/modprobe.d/CIS.conf
```

ตรวจสอบ:

```bash
lsmod | egrep "cramfs|freevxfs|jffs2|hfs|hfsplus|squashfs|udf"
```

(ควรไม่เจอ)

---

### 1.1.8 Configure `/dev/shm`

แก้ไข `/etc/fstab`:

```bash
tmpfs /dev/shm tmpfs defaults,noexec,nodev,nosuid 0 0
```

รีโหลด:

```bash
mount -o remount /dev/shm
```

---

## **1.2 Configure Software Updates**

* เปิดการอัปเดตอัตโนมัติ

```bash
apt-get install unattended-upgrades apt-listchanges -y
dpkg-reconfigure --priority=low unattended-upgrades
```

ตรวจสอบไฟล์ `/etc/apt/apt.conf.d/50unattended-upgrades`

---

## **1.5 Additional Process Hardening**

* เปิด ASLR:

```bash
echo "kernel.randomize_va_space = 2" >> /etc/sysctl.conf
sysctl -p
```

* ปิด core dumps:

```bash
echo "* hard core 0" >> /etc/security/limits.conf
```

* `/etc/systemd/coredump.conf`:

```
Storage=none
ProcessSizeMax=0
```

---

## **1.7 Command Line Warning Banners**

ไฟล์:

* `/etc/issue`
* `/etc/issue.net`
* `/etc/motd`

ตัวอย่าง:

```
Authorized uses only. All activity may be monitored and reported.
```

---

## **1.8 GNOME Display Manager**

### 1.8.1 Ensure GNOME Display Manager is removed

```bash
apt purge gdm3 -y
```

---

## **2.1 Configure Time Synchronization**

### 2.1.3 Configure `systemd-timesyncd`

```bash
apt install systemd-timesyncd -y
systemctl enable systemd-timesyncd --now
```

แก้ไข `/etc/systemd/timesyncd.conf`

```
[Time]
NTP=pool.ntp.org
FallbackNTP=ntp.ubuntu.com
```

---

## **2.2 Special Purpose Services (all)**

ปิดบริการที่ไม่จำเป็น เช่น `avahi-daemon`, `cups`, `rpcbind`

```bash
systemctl disable avahi-daemon --now
systemctl disable cups --now
systemctl disable rpcbind --now
```

---

## **2.3 Service Clients (all)**

ตรวจสอบ client tools เช่น `telnet`, `ftp`, `rsh`, `ldap-utils`, `nis`
ถอนการติดตั้ง:

```bash
apt purge telnet ftp rsh-client nis ldap-utils -y
```

---

## **3.1 Disable unused network protocols and devices**

แก้ `/etc/modprobe.d/CIS.conf`

```bash
echo "install dccp /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install sctp /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install rds /bin/true" >> /etc/modprobe.d/CIS.conf
echo "install tipc /bin/true" >> /etc/modprobe.d/CIS.conf
```

---

## **3.2 Network Parameters (Host Only)**

แก้ `/etc/sysctl.conf`

```conf
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
```

```bash
sysctl -p
```

---

## **3.4 Uncommon Network Protocols**

ปิด IPv6 ถ้าไม่ได้ใช้งาน:

```bash
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
```

---

## **3.5 Firewall Configuration**

### 3.5.1 Configure UFW

```bash
apt install ufw -y
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
```

---

## **4.1 Configure System Accounting (auditd)**

### 4.1.1 Ensure auditing is enabled

```bash
apt install auditd audispd-plugins -y
systemctl enable auditd --now
```

### 4.1.2 Configure Data Retention

แก้ `/etc/audit/auditd.conf`

```
max_log_file = 50
space_left_action = email
action_mail_acct = root
admin_space_left_action = halt
```

---

## **4.2 Configure Logging**

### 4.2.1 Configure journald

แก้ `/etc/systemd/journald.conf`

```
Storage=persistent
Compress=yes
ForwardToSyslog=yes
```

### 4.2.2 Configure rsyslog

```bash
apt install rsyslog -y
systemctl enable rsyslog --now
```

---

## **5.1 Time-based job schedulers**

* Restrict access:

```bash
rm -f /etc/cron.deny
rm -f /etc/at.deny
echo "root" > /etc/cron.allow
echo "root" > /etc/at.allow
```

---

## **5.2 Configure SSH Server**

ไฟล์ `/etc/ssh/sshd_config`

```conf
PermitRootLogin no
PasswordAuthentication no
X11Forwarding no
MaxAuthTries 4
ClientAliveInterval 300
ClientAliveCountMax 2
AllowTcpForwarding no
```

```bash
systemctl restart sshd
```

---

## **5.3 Configure sudo**

```bash
apt install sudo -y
echo "Defaults use_pty" >> /etc/sudoers
echo "Defaults logfile=\"/var/log/sudo.log\"" >> /etc/sudoers
```

---

## **5.4 Configure PAM**

ไฟล์ `/etc/security/pwquality.conf`

```
minlen = 14
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```

---

## **5.5 User Accounts and Environment**

* Lock system accounts:

```bash
for user in lp sync shutdown halt uucp operator games gopher; do
  usermod -L $user
done
```

* ตรวจสอบ `umask`:

```bash
echo "umask 027" >> /etc/profile
```

---

## **6.1 System File Permissions**

ตรวจสอบ:

```bash
stat /etc/passwd /etc/shadow /etc/group
```

แก้ไข permission:

```bash
chmod 644 /etc/passwd
chmod 640 /etc/shadow
chmod 644 /etc/group
```

---

## **6.2 Local User and Group Settings**

* ตรวจสอบ duplicate UID/GID

```bash
awk -F: '{print $3}' /etc/passwd | sort | uniq -d
awk -F: '{print $3}' /etc/group | sort | uniq -d
```

---

📌 ทั้งหมดนี้คือ **Step การ Harden ตาม CIS 22.04 LTS v1.0.0**
สามารถเอาไปทำ **Ansible Role หรือ Bash Script** ได้ต่อ

---
