# **Hardening Linux Ubuntu 24.04 (Noble Numbat)** 

คือการเสริมความปลอดภัยของระบบให้แข็งแกร่ง ป้องกันการโจมตีและลดความเสี่ยงจากการถูกบุกรุก โดยมีแนวทางหลัก ๆ ที่ควรทำดังนี้ 👇

---

## 🧱 1. ระบบพื้นฐาน (System Base Hardening)

### ✅ อัปเดตระบบให้ล่าสุด

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove --purge -y
```

> ปิดช่องโหว่จากแพ็กเกจที่ล้าสมัย

### ✅ ตั้งค่า hostname และ timezone

```bash
sudo hostnamectl set-hostname secure-host
sudo timedatectl set-timezone Asia/Bangkok
```

### ✅ ปิดบริการที่ไม่ใช้

ตรวจสอบ service ที่รันอยู่:

```bash
sudo systemctl list-unit-files --type=service --state=enabled
```

ปิด service ที่ไม่จำเป็น เช่น CUPS, Avahi:

```bash
sudo systemctl disable --now cups avahi-daemon
```

---

## 🔐 2. การรักษาความปลอดภัยของบัญชีผู้ใช้ (User & Authentication)

### ✅ จำกัดสิทธิ์ root

* ปิดการ SSH ด้วย root:

  ```bash
  sudo nano /etc/ssh/sshd_config
  ```

  แก้ค่า:

  ```
  PermitRootLogin no
  ```

  แล้ว restart SSH:

  ```bash
  sudo systemctl restart ssh
  ```

### ✅ สร้างผู้ใช้เฉพาะ

```bash
sudo adduser devops
sudo usermod -aG sudo devops
```

### ✅ ตั้งค่า Password Policy

ติดตั้ง `libpam-pwquality`

```bash
sudo apt install libpam-pwquality -y
```

แก้ไฟล์ `/etc/security/pwquality.conf` เช่น:

```
minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```

---

## 🧱 3. SSH Hardening

### ✅ ตั้งค่า SSH Security

ไฟล์ `/etc/ssh/sshd_config`:

```
Port 2222
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers devops
```

รีสตาร์ท:

```bash
sudo systemctl restart ssh
```

---

## 🧯 4. Firewall & Network Hardening

### ✅ เปิดใช้งาน UFW

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp   # หรือพอร์ต SSH ที่ตั้งไว้
sudo ufw enable
```

### ✅ ตรวจสอบ open ports

```bash
sudo ss -tuln
```

### ✅ เปิดใช้งาน `fail2ban`

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

---

## 🧩 5. File & Permission Hardening

### ✅ ตรวจสอบสิทธิ์ไฟล์สำคัญ

```bash
sudo chmod 600 /etc/ssh/ssh_host_*
sudo chmod 600 /etc/shadow
sudo chmod 644 /etc/passwd
```

### ✅ ใช้ immutable bit กับไฟล์สำคัญ

```bash
sudo chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

---

## 🧰 6. Logging & Monitoring

### ✅ เปิด Auditd

```bash
sudo apt install auditd audispd-plugins -y
sudo systemctl enable --now auditd
```

ตรวจสอบ log:

```bash
sudo aureport --summary
```

### ✅ ติดตั้งและตั้งค่า syslog หรือ rsyslog

```bash
sudo apt install rsyslog -y
sudo systemctl enable --now rsyslog
```

---

## 🧠 7. Kernel & Sysctl Hardening

แก้ไฟล์ `/etc/sysctl.conf`:

```
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.all.disable_ipv6 = 1
kernel.randomize_va_space = 2
```

แล้ว apply:

```bash
sudo sysctl -p
```

---

## 🧩 8. การป้องกัน Malware / Rootkit

```bash
sudo apt install chkrootkit rkhunter clamav -y
sudo rkhunter --update
sudo rkhunter --checkall
sudo freshclam
sudo clamscan -r /home
```

---

## 📊 9. การตรวจสอบ Security Report อัตโนมัติ

ติดตั้ง **Lynis**:

```bash
sudo apt install lynis -y
sudo lynis audit system
```

> รายงานจะอยู่ใน `/var/log/lynis.log`

---

## ⚙️ 10. อัตโนมัติและการดูแลระยะยาว

* ใช้ `unattended-upgrades`:

  ```bash
  sudo apt install unattended-upgrades -y
  sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```
* ทำ Snapshot หรือ Backup system image ก่อนปรับแก้ใหญ่
* ใช้เครื่องมือ Scan เช่น OpenSCAP หรือ CIS-CAT เพื่อตรวจเทียบกับ CIS Benchmark

---

หากคุณต้องการ ผมสามารถสร้าง **Shell Script “Ubuntu 24.04 Hardening Automation”** ที่รวมทุกขั้นตอนด้านบนให้รันอัตโนมัติได้เลย
ต้องการให้ผมสร้างสคริปต์นั้นไหมครับ?
