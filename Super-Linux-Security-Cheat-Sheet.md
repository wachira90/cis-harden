# **Linux Security Cheat Sheet** 

สรุปคำสั่งและวิธีการตั้งค่าความปลอดภัยสำหรับเซิร์ฟเวอร์ Linux (Server Hardening) ที่เน้นให้นำไปใช้งานหรือคัดลอกคำสั่งไปรันได้ทันที โดยแบ่งเนื้อหาออกเป็น 2 ส่วนหลักๆ ดังนี้

### **ส่วนที่ 1: การตั้งค่าพื้นฐานที่จำเป็น (The Essentials)**

เป็นสิ่งที่ควรทำทันทีหลังจากเปิดใช้งานเซิร์ฟเวอร์ใหม่:

1. **System Updates:** อัปเดตแพ็กเกจระบบปฏิบัติการให้ใหม่ล่าสุดอยู่เสมอเพื่ออุดช่องโหว่ (ใช้คำสั่ง `apt` หรือ `yum`)
2. **User Management:** ไม่ควรใช้บัญชี `root` ทำงานโดยตรง ให้สร้างผู้ใช้ใหม่และมอบสิทธิ์ Sudo ให้แทน
3. **Password Policies:** บังคับให้ผู้ใช้เปลี่ยนรหัสผ่านตามรอบเวลา และล็อกบัญชีหากไม่มีการใช้งานนานเกินไป
4. **Locking Root Account:** ล็อกบัญชี `root` เพื่อป้องกันการล็อกอินเข้ามาโดยตรง
5. **Basic SSH Config:** ตั้งค่า SSH ให้ปลอดภัยขึ้น เช่น เปลี่ยนพอร์ตเริ่มต้น, ปิดการล็อกอินด้วย `root`, ห้ามใช้รหัสผ่านว่างเปล่า และจำกัดจำนวนครั้งที่ล็อกอินพลาด
6. **Firewall Basics:** ตั้งค่า Firewall ปิดการเชื่อมต่อขาเข้าทั้งหมด (Default Deny) แล้วค่อยเปิดอนุญาตเฉพาะพอร์ตที่จำเป็น เช่น SSH, HTTP (โดยใช้ UFW หรือ Firewalld)
7. **Disabling Services:** ปิด Services ที่ไม่ได้ใช้งานทิ้งไปเพื่อลดช่องโหว่ของระบบ
8. **File Permissions:** กำหนดสิทธิ์ไฟล์และโฟลเดอร์ให้ถูกต้อง (เช่น 600 สำหรับ Private Keys ที่มีความสำคัญมาก)
9. **Time Synchronization:** เปิดใช้งาน NTP เพื่อให้เวลาของเซิร์ฟเวอร์ตรงกัน ซึ่งสำคัญมากต่อการตรวจสอบความถูกต้องของ Log
10. **Checking Active Sessions:** หมั่นตรวจสอบว่ามีใครล็อกอินอยู่ หรือมีการพยายามล็อกอินแบบเดารหัสผ่าน (Brute-force) ล้มเหลวหรือไม่
11. **Shell History Hygiene:** ล้างประวัติคำสั่ง (Command History) เพื่อป้องกันไม่ให้ข้อมูลสำคัญ เช่น รหัสผ่านที่อาจเผลอพิมพ์ลงไป หลุดรอดไปได้

### **ส่วนที่ 2: การรักษาความปลอดภัยขั้นสูง (Advanced Hardening)**

สำหรับการเตรียมเซิร์ฟเวอร์ระดับ Production ให้มีความปลอดภัยสูงสุด:
12. **Mastering Sudoers:** ใช้ `visudo` ในการจัดการสิทธิ์อย่างละเอียด เช่น การจำกัดให้บางผู้ใช้รันได้เฉพาะบางคำสั่งเท่านั้น
13. **Advanced SSH Security:** ปิดการใช้รหัสผ่านโดยสิ้นเชิงและบังคับให้ใช้ SSH Keys เท่านั้น พร้อมทั้งแนะนำให้เปิดใช้งานระบบ 2FA (เช่น Google Authenticator)
14. **Intrusion Prevention (Fail2Ban):** ติดตั้ง Fail2Ban เพื่อช่วยบล็อก IP ที่พยายามเดารหัสผ่านผิดซ้ำๆ โดยอัตโนมัติ
15. **Immutable Files:** ใช้คำสั่ง `chattr +i` ล็อกไฟล์สำคัญของระบบ (เช่น `/etc/passwd`) ไม่ให้ใครลบหรือแก้ไขได้ แม้แต่ผู้ที่มีสิทธิ์ root ก็ตาม
16. **Access Control Lists (ACLs):** ใช้กำหนดสิทธิ์การเข้าถึงไฟล์แบบละเอียดรายบุคคล ซึ่งยืดหยุ่นกว่าการใช้ chmod ปกติ
17. **Port & Socket Auditing:** ตรวจสอบพอร์ตทั้งหมดที่เปิดอยู่บนระบบด้วยคำสั่งอย่าง `ss` หรือ `lsof` เพื่อหา Service ที่ไม่ได้รับอนุญาต
18. **Kernel Hardening:** ปรับแต่งไฟล์ `sysctl.conf` เพื่อป้องกันการโจมตีระดับเครือข่าย เช่น ปิดการตอบกลับ Ping (Stealth Mode) และป้องกันการปลอมแปลง IP (IP Spoofing)
19. **Process Limits:** จำกัดการใช้ทรัพยากร (เช่น จำนวน Process หรือไฟล์ที่เปิดได้) ของแต่ละผู้ใช้เพื่อป้องกันการโจมตีแบบ DoS
20. **Log Analysis & Forensics:** ใช้ `journalctl` ตรวจสอบและวิเคราะห์หาความผิดปกติจาก Log ของระบบ
21. **Rootkit Scanning:** ติดตั้งเครื่องมืออย่าง `rkhunter` เพื่อสแกนหาซอฟต์แวร์ประสงค์ร้ายและ Backdoor แบบอัตโนมัติ
22. **Cron Job Security:** ตรวจสอบระบบตั้งเวลาการทำงาน (Cron jobs) อย่างละเอียด เพราะมัลแวร์มักจะซ่อนคำสั่งรันตัวเองไว้ในนี้
23. **Legal Warning Banners:** ใส่ข้อความเตือนทางกฎหมายก่อนล็อกอิน SSH เพื่อขู่ผู้ไม่ประสงค์ดีและเตรียมพร้อมในแง่กฎหมาย


# รายละเอียดการตั้งค่าและคำสั่งของแต่ละข้อ 

อ้างอิงจากบทความที่คุณส่งมา คุณสามารถคัดลอกคำสั่ง (Copy-paste) ไปปรับใช้กับเซิร์ฟเวอร์ของคุณได้เลย

## ส่วนที่ 1: การตั้งค่าพื้นฐานที่จำเป็น (The Essentials)

ควรทำทันทีหลังจากเปิดใช้งานเซิร์ฟเวอร์ใหม่

**1. System Updates (อัปเดตระบบ)**
อย่าปล่อยให้แพ็กเกจล้าสมัย อัปเดตทันทีเพื่ออุดช่องโหว่

* **Debian/Ubuntu:**
```bash

```



sudo apt update && sudo apt upgrade -y

```
*   **RHEL/CentOS/AlmaLinux:**
    ```bash
sudo yum update -y

```

**2. User Management (จัดการผู้ใช้งาน)**
สร้างผู้ใช้ใหม่สำหรับทำงานแทนการใช้ `root`

```bash
# สร้างผู้ใช้ชื่อ 'oz' พร้อมสร้างโฟลเดอร์ Home และตั้งค่า Shell เป็น Bash
sudo useradd -m -s /bin/bash oz

# ตั้งรหัสผ่านให้ 'oz'
sudo passwd oz

# มอบสิทธิ์ Sudo ให้ (Debian/Ubuntu ใช้กลุ่ม sudo, RHEL/CentOS ใช้กลุ่ม wheel)
sudo usermod -aG sudo oz   # สำหรับ Debian/Ubuntu
sudo usermod -aG wheel oz  # สำหรับ RHEL/CentOS

```

**3. Password Policies (นโยบายรหัสผ่าน)**
บังคับให้เปลี่ยนรหัสผ่านตามรอบเวลาที่กำหนด

```bash
# ดูสถานะรหัสผ่านปัจจุบันของ user 'oz'
sudo chage -l oz

# บังคับให้เปลี่ยนรหัสผ่านทุกๆ 90 วัน
sudo chage -M 90 oz

# ล็อกบัญชีหากไม่มีการใช้งานติดต่อกัน 30 วัน
sudo chage -I 30 oz

```

**4. Locking Root Account (ล็อกบัญชี Root)**
หลังจากมี User ที่มีสิทธิ์ Sudo แล้ว ให้ล็อกการเข้าสู่ระบบของ `root` โดยตรง

```bash
# ล็อกบัญชี root
sudo passwd -l root

# (ถ้าจำเป็นต้องปลดล็อก ค่อยใช้คำสั่งนี้)
# sudo passwd -u root

```

**5. Basic SSH Config (ตั้งค่าความปลอดภัย SSH)**
แก้ไขไฟล์ `/etc/ssh/sshd_config` (แนะนำให้แบ็คอัปไฟล์เดิมไว้ก่อนด้วย `cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`)

```bash
sudo nano /etc/ssh/sshd_config

```

ค้นหาและแก้ไขค่าต่อไปนี้:

```text
Port 2222                   # เปลี่ยนพอร์ตเริ่มต้นจาก 22
PermitRootLogin no          # ห้าม root ล็อกอินโดยตรง (สำคัญมาก!)
PasswordAuthentication no   # ปิดการล็อกอินด้วยรหัสผ่าน (ใช้ในกรณีที่ตั้งค่า SSH Key แล้ว)
PermitEmptyPasswords no     # ไม่อนุญาตให้ใช้รหัสผ่านว่างเปล่า
MaxAuthTries 3              # ตัดการเชื่อมต่อหากใส่รหัสผิดเกิน 3 ครั้ง
LoginGraceTime 60           # ตัดการเชื่อมต่อหากล็อกอินไม่สำเร็จภายใน 60 วินาที

```

บันทึกไฟล์แล้วรีสตาร์ท SSH:

```bash
sudo systemctl restart sshd

```

**6. Firewall Basics (ตั้งค่าไฟร์วอลล์)**
ตั้งกฎปิดทั้งหมดก่อน แล้วค่อยเปิดเฉพาะพอร์ตที่ใช้

* **UFW (แนะนำสำหรับ Ubuntu/Debian):**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp      # พอร์ต SSH ที่เราเพิ่งเปลี่ยน (ถ้าใช้พอร์ต 22 ก็ใส่ 22)
sudo ufw allow 80/tcp        # ถ้ามี Web Server
sudo ufw allow 443/tcp       # ถ้ามี Web Server (HTTPS)
sudo ufw enable

```



```
*   **Firewalld (สำหรับ RHEL/CentOS):**
    ```bash
    sudo firewall-cmd --permanent --add-port=2222/tcp
    sudo firewall-cmd --reload

```

**7. Disabling Services (ปิด Service ที่ไม่จำเป็น)**
ตรวจสอบและปิดการทำงานของ Service ที่ไม่ได้ใช้เพื่อลดช่องโหว่

```bash
# ดูรายการ Service ที่ทำงานอยู่ทั้งหมด
systemctl list-unit-files --state=enabled

# ตัวอย่างการปิด Service ของ postfix (สมมติว่าไม่ได้ใช้)
sudo systemctl stop postfix
sudo systemctl disable postfix

```

**8. File Permissions 101 (กำหนดสิทธิ์ไฟล์)**
มาตรฐานการให้สิทธิ์เพื่อป้องกันการแก้ไขไฟล์จากผู้ที่ไม่เกี่ยวข้อง

```bash
chmod 755 script.sh   # ไฟล์สคริปต์ / โฟลเดอร์ (เจ้าของอ่านเขียนรันได้ คนอื่นอ่านและรันได้)
chmod 644 config.yaml # ไฟล์ตั้งค่า (เจ้าของอ่านเขียนได้ คนอื่นอ่านได้อย่างเดียว)
chmod 600 id_rsa      # Private Key (สำคัญมาก: ให้เจ้าของอ่านและเขียนได้คนเดียวเท่านั้น)

```

**9. Time Synchronization (ซิงค์เวลาเซิร์ฟเวอร์)**
สำคัญมากสำหรับการวิเคราะห์ Log

```bash
# เปิดใช้งานการซิงค์เวลา NTP
sudo timedatectl set-ntp on

# ตรวจสอบสถานะ
timedatectl status

```

**10. Checking Active Sessions (ตรวจสอบการล็อกอิน)**
ตรวจสอบว่ามีใครใช้งานระบบอยู่บ้าง

```bash
w                 # ดูว่าใครล็อกอินอยู่ตอนนี้
last -n 10        # ดูประวัติคนล็อกอิน 10 รายการล่าสุด
sudo lastb -n 10  # ดูประวัติคนที่พยายามล็อกอินแต่ล้มเหลว (เช็คการเดารหัสผ่าน)

```

**11. Shell History Hygiene (ล้างประวัติคำสั่ง)**
ป้องกันรหัสผ่านหรือข้อมูลความลับตกค้างในประวัติการพิมพ์คำสั่ง

```bash
history -c                       # ล้างประวัติของเซสชันปัจจุบัน
cat /dev/null > ~/.bash_history  # ลบประวัติทั้งหมดในไฟล์
export HISTSIZE=0                # ปิดการบันทึกประวัติสำหรับเซสชันนี้

```

---

## ส่วนที่ 2: การรักษาความปลอดภัยขั้นสูง (Advanced Hardening)

สำหรับการทำเซิร์ฟเวอร์ระดับ Production

**12. Mastering Sudoers (การจัดการสิทธิ์ Sudo เชิงลึก)**
ใช้ `sudo visudo` เสมอ ห้ามแก้ไขไฟล์ `/etc/sudoers` โดยตรงเพื่อป้องกันเซิร์ฟเวอร์พังจากไวยากรณ์ผิด

```bash
sudo visudo

```

ตัวอย่างการตั้งค่าในไฟล์:

```text
# ให้ user 'deploy' รันคำสั่งทั้งหมดได้โดยไม่ต้องใช้รหัสผ่าน (ใช้กับระบบ Automation)
deploy ALL=(ALL) NOPASSWD: ALL

# ให้ user 'junior' สั่งรีสตาร์ท Nginx ได้อย่างเดียว ห้ามทำอย่างอื่น
junior ALL=(ALL) /usr/bin/systemctl restart nginx

# ให้ทุกคนในกลุ่ม 'sysadmin' มีสิทธิ์ Sudo เต็มรูปแบบ
%sysadmin ALL=(ALL) ALL

```

**13. Advanced SSH Security (เสริมความปลอดภัย SSH)**
บังคับใช้ SSH Keys และ 2FA (อย่าลืมคัดลอก Key ตัวเองเข้าเซิร์ฟเวอร์ด้วย `ssh-copy-id` ก่อนทำขั้นตอนนี้)

```bash
# แก้ไขไฟล์ /etc/ssh/sshd_config 
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no

```

ติดตั้ง 2FA (Google Authenticator) เพิ่มเติม (ตัวอย่างบน RHEL/CentOS):

```bash
sudo dnf install libpam-google-authenticator google-authenticator
# จากนั้นพิมพ์คำสั่ง google-authenticator และทำตามขั้นตอนบนหน้าจอ

```

**14. Intrusion Prevention (ติดตั้ง Fail2Ban)**
ระบบป้องกันการโจมตีแบบ Brute-force อัตโนมัติ

```bash
sudo dnf install fail2ban -y

# คัดลอกไฟล์ตั้งค่าก่อนแก้ไข (อย่าแก้ที่ jail.conf ตรงๆ)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

```

เพิ่มการตั้งค่าสำหรับ SSH ในไฟล์ `jail.local`:

```ini
[sshd]
enabled = true
port = ssh        # หรือใส่พอร์ตที่เปลี่ยนไปเช่น 2222
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3      # จำนวนครั้งที่ยอมให้ผิดพลาด
bantime = 1h      # แบน 1 ชั่วโมง

```

สั่งเริ่มระบบ:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

```

**15. Immutable Files (ล็อกไฟล์สำคัญไม่ให้ลบ/แก้)**
ใช้คำสั่ง `chattr` เพื่อสร้างเกราะป้องกันไฟล์ระบบ แม้แต่ root ก็ลบไม่ได้

```bash
sudo chattr +i /etc/passwd    # ล็อกไฟล์
sudo chattr +i /etc/shadow

lsattr /etc/passwd            # ตรวจสอบว่ามีตัวอักษร 'i' ติดอยู่หรือไม่
rm /etc/passwd                # ทดสอบลบ (ระบบจะแจ้ง Operation not permitted)

sudo chattr -i /etc/passwd    # วิธีปลดล็อกเมื่อต้องการแก้ไข

```

**16. Access Control Lists (ACLs - กำหนดสิทธิ์แบบยืดหยุ่น)**
เมื่อการตั้งสิทธิ์แบบ `chmod` ทั่วไปไม่ตอบโจทย์

```bash
# อนุญาตให้ user ชื่อ 'john' มีสิทธิ์อ่านเขียนและรันไฟล์นี้ได้เต็มที่ (โดยไม่สนว่าใครเป็นเจ้าของไฟล์)
setfacl -m u:john:rwx /var/www/html/index.php

# ดูสิทธิ์ ACL ทั้งหมดของไฟล์
getfacl /var/www/html/index.php

# ลบสิทธิ์ของ john ออกจากไฟล์นี้
setfacl -x u:john /var/www/html/index.php

```

**17. Port & Socket Auditing (ตรวจสอบพอร์ตและเครือข่าย)**
ตรวจดูว่ามีแอปพลิเคชันแปลกปลอมแอบเปิดพอร์ตทิ้งไว้หรือไม่

```bash
sudo ss -tulpn         # ดูพอร์ต TCP/UDP ทั้งหมดที่เปิดรอรับการเชื่อมต่อ พร้อมชื่อ Process
sudo lsof -i :8080     # ดูว่าใครกำลังใช้งานพอร์ต 8080 อยู่
sudo nethogs eth0      # ดูแบนด์วิดท์ที่ใช้แบบเรียลไทม์แยกตาม Process (ต้องติดตั้ง nethogs ก่อน)

```

**18. Kernel Hardening (ปรับจูน Kernel ป้องกันการโจมตีเครือข่าย)**
เข้าไปแก้ไขไฟล์ควบคุม Kernel

```bash
sudo nano /etc/sysctl.conf

```

เพิ่มหรือแก้ไขค่าเหล่านี้:

```ini
# ปิดการ Forward แพ็กเกจ (ถ้าไม่ได้ทำเซิร์ฟเวอร์เป็น Router)
net.ipv4.ip_forward = 0

# ปิดการตอบกลับ Ping (ทำให้แฮกเกอร์สแกนหาเซิร์ฟเวอร์ยากขึ้น)
net.ipv4.icmp_echo_ignore_all = 1

# ป้องกันการปลอมแปลง IP (IP Spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# ปิดการใช้งาน IPv6 (หากไม่ได้ใช้)
net.ipv6.conf.all.disable_ipv6 = 1

```

ใช้คำสั่งนี้เพื่อให้การตั้งค่ามีผลทันที:

```bash
sudo sysctl -p

```

**19. Process Limits (จำกัดการใช้ทรัพยากร)**
ป้องกันเซิร์ฟเวอร์ค้างจากการใช้ทรัพยากรเกินขนาด (เช่น DoS Attack)

```bash
sudo nano /etc/security/limits.conf

```

เพิ่มค่าเหล่านี้เข้าไป:

```text
# จำกัด user 'oz' ให้รัน Process ได้สูงสุด 50 ตัว
oz hard nproc 50

# เพิ่มขีดจำกัดการเปิดไฟล์พร้อมกันของ user 'postgres' (เหมาะสำหรับฐานข้อมูล)
postgres soft nofile 4096
postgres hard nofile 10240

```

**20. Log Analysis & Forensics (การวิเคราะห์ Log)**
ตรวจสอบบันทึกของระบบเพื่อค้นหาความผิดปกติ

```bash
# ดู Log การเข้าสู่ระบบแบบเรียลไทม์ (Debian/Ubuntu)
tail -f /var/log/auth.log

# ดู Log ของ SSH ย้อนหลัง 1 ชั่วโมง (ผ่านระบบ Systemd Journal)
journalctl -u sshd --since "1 hour ago"

# กรองดูเฉพาะ Log ที่เป็น Error หรือ Critical เท่านั้น (ตั้งแต่เปิดเครื่องรอบนี้)
journalctl -p err -b

```

**21. Rootkit Scanning (สแกนหาซอฟต์แวร์ฝังตัวอันตราย)**
เครื่องมือตรวจสอบมัลแวร์และ Backdoor ระดับราก

```bash
# ติดตั้ง rkhunter
sudo dnf install rkhunter -y

# อัปเดตฐานข้อมูลมัลแวร์
sudo rkhunter --propupd

# สั่งให้ระบบทำการตรวจสอบ (ใช้เวลาสักครู่)
sudo rkhunter --check

```

**22. Cron Job Security (ความปลอดภัยการตั้งเวลาทำงาน)**
แฮกเกอร์มักฝังสคริปต์ในระบบตั้งเวลา (Cron) เพื่อให้มัลแวร์รันตัวเองซ้ำๆ

```bash
# ดูรายการตั้งเวลาทำงานของ User ปัจจุบัน
crontab -l

# ตรวจสอบโฟลเดอร์ตั้งเวลาทำงานของระบบว่ามีไฟล์อะไรแปลกปลอมไหม
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
cat /etc/crontab

```

**23. Legal Warning Banners (ข้อความเตือนทางกฎหมาย)**
แสดงข้อความข่มขู่เมื่อมีคนพยายามล็อกอิน เพื่อผลทางจิตวิทยาและกฎหมาย

```bash
# สร้าง/แก้ไขไฟล์แบนเนอร์
sudo nano /etc/issue.net

```

เพิ่มข้อความเช่น: *"UNAUTHORIZED ACCESS PROHIBITED. ALL ACTIVITY IS LOGGED."*

จากนั้นไปเปิดใช้งานในไฟล์ SSH:

```bash
sudo nano /etc/ssh/sshd_config

```

ค้นหาและนำเครื่องหมาย `#` หน้าคำว่า `Banner` ออก แล้วใส่พาธไฟล์เข้าไป:

```text
Banner /etc/issue.net

```
