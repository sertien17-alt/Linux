# Samba Active Directory Domain Controller on AWS (Ubuntu)

This guide explains how to install and configure a Samba Active Directory Domain Controller (AD DC) on Ubuntu in AWS, and how to configure a second Linux server as a domain client.

---

# Phase 1 — Configure Hostname and Static IP

## Set Hostname


sudo hostnamectl set-hostname ls204


Edit hosts file:


sudo nano /etc/hosts



Example:


127.0.0.1       localhost
192.168.1.204   ls04.lab04.lan ls04


Reboot:

sudo reboot

---

# Phase 2 — Install Required Packages

Update repositories:

sudo apt update


Install Samba and dependencies:

sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools


When prompted for Kerberos Realm, enter:

LAB04.LAN

![Linux Course](Imagenes/Sprint5Image/5_1.png)

![Linux Course](Imagenes/Sprint5Image/5_2.png)

![Linux Course](Imagenes/Sprint5Image/5_3.png)

# Phase 3 — Disable Classic Samba Services

Stop and disable services not used by AD DC:

sudo systemctl stop smbd nmbd winbind
sudo systemctl disable smbd nmbd winbind

![Linux Course](Imagenes/Sprint5Image/5_4.png)

Enable AD DC service:


sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc


---

# Phase 4 — Backup Default Samba Configuration


sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak


---

# Phase 5 — Provision Samba Active Directory

Run provisioning:


sudo samba-tool domain provision


Use the following values:

- Realm: `LAB04.LAN`
- Domain: `LAB04`
- Server Role: `dc`
- DNS Backend: `SAMBA_INTERNAL`
- Administrator password: (define strong password)

---

## Configure Kerberos

Backup original file:


sudo mv /etc/krb5.conf /etc/krb5.conf.orig


Copy Samba-generated config:


sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf


(Optional verification)


sudo nano /etc/krb5.conf


No modifications are required.

---

# Phase 6 — Disable systemd-resolved (DNS Fix)

Disable service:


sudo systemctl disable --now systemd-resolved

Remove symbolic link:

sudo unlink /etc/resolv.conf


Create new resolv.conf:

sudo nano /etc/resolv.conf

![Linux Course](Imagenes/Sprint5Image/5_6.png)


Add:


nameserver 127.0.0.1
search LAB04.LAN


Make file immutable:


sudo chattr +i /etc/resolv.conf


---

# Phase 7 — Start Domain Controller

Start service:


sudo systemctl start samba-ad-dc

Check status:


sudo systemctl status samba-ad-dc


![Linux Course](Imagenes/Sprint5Image/5_5.png)

---

# Phase 8 — Validation

Test Kerberos authentication:


kinit administrator@LAB04.LAN

![Linux Course](Imagenes/Sprint5Image/5_7.png)


Verify ticket:


klist

![Linux Course](Imagenes/Sprint5Image/5_8.png)


If successful, the Domain Controller is operational.

---

# Configure a Second Linux Server as a Domain Client


su - administrator@LAB04.LAN


If login works, the client server is successfully joined to the domain.

---


## 1. Set the hostname of the client machine:
$ sudo hostnamectl set-hostname cli-ssd

![Linux Course](Imagenes/Sprint2Image/2_2.png)


## 2. Hosts File Configuration

Edit the hosts file to add server and domain resolution:
$ sudo nano /etc/hosts

![Linux Course](Imagenes/Sprint2Image/2_5.png)

![Linux Course](Imagenes/Sprint2Image/2_6.png)


## 3. Connectivity and Internet Test

Verify DNS and internet access:
$ ping lab04.lan
$ ping 8.8.8.8

## 4. Enable IP Forwarding on Server

Edit sysctl configuration:
$ sudo nano /etc/sysctl.conf

Add at the end:
net.ipv4.ip_forward=1

![Linux Course](Imagenes/Sprint2Image/2_11.png)

Apply and verify:
$ sudo sysctl -p
$ sysctl net.ipv4.ip_forward

![Linux Course](Imagenes/Sprint2Image/2_12.png)

## 5. Configure NAT on Server

Enable NAT so the client can access the internet:
$ sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
$ sudo iptables -t nat -L -n -v

![Linux Course](Imagenes/Sprint2Image/2_13.png)

## 6. Time Synchronization

Server configuration with Chrony:
$ sudo apt update
$ sudo apt install chrony -y
$ sudo nano /etc/chrony/chrony.conf

![Linux Course](Imagenes/Sprint2Image/2_14.png)

Modify:
allow 0

Restart and allow NTP:
$ sudo systemctl restart chrony
$ sudo ufw allow 123/udp

Client configuration:
$ sudo nano /etc/systemd/timesyncd.conf
$ sudo systemctl restart systemd-timesyncd
$ timedatectl show-timesync --all

![Linux Course](Imagenes/Sprint2Image/2_15.png)

![Linux Course](Imagenes/Sprint2Image/2_16.png)


Alternative synchronization method:
$ sudo apt install ntpdate
$ sudo ntpdate lab04.lan
$ sudo ntpdate -q lab04.lan

![Linux Course](Imagenes/Sprint2Image/2_17.png)

## 7. Install Required Packages on Ubuntu Desktop

Install required packages:
$ sudo apt update
$ sudo apt install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind

If installation fails:
$ sudo systemctl stop unattended-upgrades
$ sudo kill -9 4363
$ sudo dpkg --configure -a
$ sudo apt install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind


![Linux Course](Imagenes/Sprint2Image/2_18.png)

## 8. Kerberos Authentication Test

Test Kerberos authentication:
$ kinit administrator@LAB04.LAN
$ klist

![Linux Course](Imagenes/Sprint2Image/2_19.png)

## 9. Kerberos Configuration

Edit Kerberos configuration:
$ sudo nano /etc/krb5.conf

Add:
dns_lookup_realm = true
dns_lookup_kdc = true

![Linux Course](Imagenes/Sprint2Image/2_20.png)

## 10. Samba Configuration

Backup and recreate Samba configuration:
$ sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.initial
$ sudo nano /etc/samba/smb.conf

![Linux Course](Imagenes/Sprint2Image/2_21.png)

Restart required services:
$ sudo systemctl restart smbd nmbd
$ sudo systemctl stop samba-ad-dc
$ sudo systemctl enable smbd nmbd

![Linux Course](Imagenes/Sprint2Image/2_22.png)

## 11. Join Ubuntu Desktop to Samba Active Directory

Join the domain:
$ sudo net ads join -U administrator

![Linux Course](Imagenes/Sprint2Image/2_23.png)

Verify on server:
$ sudo samba-tool computer list

![Linux Course](Imagenes/Sprint2Image/2_24.png)

## 12. Configure Authentication with Active Directory

Edit NSS configuration:
$ sudo nano /etc/nsswitch.conf

![Linux Course](Imagenes/Sprint2Image/2_25.png)

Restart Winbind and verify users:
$ sudo systemctl restart winbind
$ wbinfo -u
$ wbinfo -g
$ getent passwd | grep administrator
$ id administrator

![Linux Course](Imagenes/Sprint2Image/2_26.png)

## 13. PAM Configuration for Home Directory Creation

Enable automatic home directory creation:
$ sudo pam-auth-update

![Linux Course](Imagenes/Sprint2Image/2_27.png)

Edit PAM configuration:
$ sudo nano /etc/pam.d/common-account

Add at the end:
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022

![Linux Course](Imagenes/Sprint2Image/2_28.png)

## 14. Grant Sudo Permissions to Domain Administrator

Add administrator to sudo group:
$ sudo usermod -aG sudo administrator

![Linux Course](Imagenes/Sprint2Image/2_29.png)

Login using:
administrator@lab04.lan

![Linux Course](Imagenes/Sprint2Image/2_30.png)

## 15. User and Group Management in Samba AD

Create group and users:
$ sudo samba-tool group add IT_departaments --group-scope=Universal --group-type=Security

![Linux Course](Imagenes/Sprint2Image/2_31.png)

$ sudo samba-tool user create alice

![Linux Course](Imagenes/Sprint2Image/2_32.png)

$ sudo samba-tool group addmembers IT_admins alice

![Linux Course](Imagenes/Sprint2Image/2_33.png)

## 16. Organizational Units (OU)

Create OU:
$ sudo samba-tool ou create "OU=IT_departaments,DC=lab04,DC=lan"

![Linux Course](Imagenes/Sprint2Image/2_34.png)

Move users:
$ sudo samba-tool user move alice "OU=IT_departaments,DC=lab04,DC=lan"
$ sudo samba-tool user move bob "OU=Students,DC=lab04,DC=lan"
$ sudo samba-tool user move charlie "OU=HR_Department,DC=lab04,DC=lan"

Move groups:
$ sudo samba-tool group move IT_admins "OU=IT_departaments,DC=lab04,DC=lan"
$ sudo samba-tool group move Students "OU=Students,DC=lab04,DC=lan"
$ sudo samba-tool group move IT_departaments "OU=HR_Department,DC=lab04,DC=lan"

![Linux Course](Imagenes/Sprint2Image/2_35.png)

Verify:
$ sudo samba-tool ou list

![Linux Course](Imagenes/Sprint2Image/2_36.png)

## 17. Group Policy Objects (GPO)

Create GPO:
$ sudo samba-tool gpo create "IT_Security_Policy" -U Administrator

![Linux Course](Imagenes/Sprint2Image/2_37.png)

Link GPO to IT OU:
$ sudo samba-tool gpo setlink "OU=IT_departaments,DC=lab04,DC=lan" {6DCC05BC-2848-4F4E-89E4-44EBE1A4C823} -U Administrator


![Linux Course](Imagenes/Sprint2Image/2_38.png)

## 18. Password Security Policies

Set domain password policies:
$ sudo samba-tool domain passwordsettings set --min-pwd-length=8


![Linux Course](Imagenes/Sprint2Image/2_39.png)

$ sudo samba-tool domain passwordsettings set --account-lockout-threshold=3


![Linux Course](Imagenes/Sprint2Image/2_40.png)

$ sudo samba-tool domain passwordsettings set --account-lockout-duration=5


![Linux Course](Imagenes/Sprint2Image/2_41.png)

Verify:


![Linux Course](Imagenes/Sprint2Image/2_42.png)

## 19. Password Settings Object (PSO)

Create strict PSO:
$ sudo samba-tool domain passwordsettings pso create "PSO_IT_Estricta" 10 --account-lockout-threshold=3 --account-lockout-duration=5 --reset-account-lockout-after=5 -U Administrator


![Linux Course](Imagenes/Sprint2Image/2_43.png)

Apply PSO to IT group:
$ sudo samba-tool domain passwordsettings pso apply "PSO_IT_Estricta" "it_admins" -U Administrator

![Linux Course](Imagenes/Sprint2Image/2_44.png)

