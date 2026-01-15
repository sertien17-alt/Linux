## Sprint 1 – Lab Activity  

# Linux Server – Ubuntu + Samba Active Directory Domain Controller

**Linux Server (Ubuntu Server) + Samba AD DC**

![Linux Course](Imagenes/0.png)


## 1. Virtual Machine Creation

Create a virtual machine with the following specifications:

- **OS:** Ubuntu Server 22.04 or 24.04  
- **RAM:** 4 GB  
- **CPUs:** 2  
- **Disk:** 40 GB  
- **Network Adapters:** 2  



## 2. VirtualBox Network Configuration

### Adapter 1 – INTERNET
- **Enabled**
- **Attached to:** Bridged Adapter  
- **Purpose:**  
  - Internet access  
  - Real IP from the physical network  

**Server IP:**  
172.30.20.39

![Network Configuration](Imagenes/1.png)

### Adapter 2 – DOMAIN NETWORK
- **Enabled**
- **Attached to:** Internal Network  
- **Name:** intnet  
- **Purpose:**  
  - Internal traffic for future domain clients  

---

## 3. Ubuntu Server Installation and Network Configuration

### Server Setup
- **Hostname:** ls04  
- **Username:** Sergio  
- **Password:** admin_21  

![Network Configuration](Imagenes/2.png)

![Network Configuration](Imagenes/3.png)

## 4. Network Configuration (Netplan)

Edit Netplan configuration:

sudo nano /etc/netplan/50-cloud-init.yaml

### Bridged Adapter (Internet)

addresses: 172.30.20.39/16  
gateway4: 172.30.20.1  
DNS: 10.239.3.7, 10.239.3.8  

### Internal Network Adapter (Domain)

addresses: 192.168.10.37/24  
DNS: 127.0.0.1  

Apply changes:

sudo netplan apply

![Network Configuration](Imagenes/4.png)

## 5. Why Use a FQDN?

- Active Directory depends on Kerberos
- Kerberos requires a Fully Qualified Domain Name (FQDN)
- Samba DNS registers services using FQDN
- Without it, samba-tool domain provision may fail

---

## 6. Configure Hostname and Hosts File

Set hostname:

sudo hostnamectl set-hostname ls04


![Network Configuration](Imagenes/5.png)

Edit hosts file:

sudo nano /etc/hosts

![Network Configuration](Imagenes/6.png)

Add server IP and hostname.

Reboot:

sudo reboot

Verify:

ping ls04


![Network Configuration](Imagenes/7.png)


![Network Configuration](Imagenes/8.png)

## 7. Install SSH (Optional)

sudo apt install openssh-server


![Network Configuration](Imagenes/9.png)

## 8. Prepare Server for Samba AD

Disable systemd-resolved:

sudo systemctl disable --now systemd-resolved

![Network Configuration](Imagenes/10.png)

Remove symlink:

sudo unlink /etc/resolv.conf

Create resolv.conf:

sudo nano /etc/resolv.conf

![Network Configuration](Imagenes/11.png)

Make immutable:

sudo chattr +i /etc/resolv.conf

## 9. Install Samba and Dependencies

sudo apt update  
sudo apt install samba winbind smbclient krb5-user dnsutils -y


![Network Configuration](Imagenes/12.png)

![Network Configuration](Imagenes/13.png)

![Network Configuration](Imagenes/14.png)

![Network Configuration](Imagenes/15.png)

## 10. Disable Classic Samba Services

sudo systemctl stop smbd nmbd winbind  
sudo systemctl disable smbd nmbd winbind  

Enable Samba AD DC:

sudo systemctl unmask samba-ad-dc  
sudo systemctl enable samba-ad-dc  

![Network Configuration](Imagenes/16.png)

## 11. Backup Samba Configuration

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak


## 12. Provision Samba Active Directory

sudo samba-tool domain provision

![Network Configuration](Imagenes/17.png)

Copy Kerberos config:

sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf



## 13. Start Samba AD DC

sudo systemctl start samba-ad-dc  
sudo systemctl status samba-ad-dc  

![Network Configuration](Imagenes/18.png)


## 14. Configure Time Synchronization (NTP)

Install Chrony:

sudo apt install chrony

Set permissions:

sudo chown root:_chrony /var/lib/samba/ntp_signd  
sudo chmod 750 /var/lib/samba/ntp_signd  

Edit Chrony config:

sudo nano /etc/chrony/chrony.conf

Add:

bindcmdaddress 192.168.10.37  
allow 192.168.10.1/24  
ntpsigndsocket /var/lib/samba/ntp_signd  

![Network Configuration](Imagenes/19.png)

Restart Chrony:

sudo systemctl restart chronyd  
sudo systemctl status chronyd  


## 15. Verify Domain Records

host -t A lab04.lan  
host -t A ls04.lab04.lan  

![Network Configuration](Imagenes/20.png)

Kerberos and LDAP:

host -t SRV _kerberos._udp.lab04.lan  
host -t SRV _ldap._tcp.lab04.lan  

![Network Configuration](Imagenes/21.png)

## 16. Verify Samba Shares

smbclient -L lab04.lan -N

![Network Configuration](Imagenes/22.png)

## 17. Final Validation – Kerberos

Authenticate:

kinit administrator@LAB04.LAN

Password:

admin_21

Verify ticket:

klist

![Network Configuration](Imagenes/23.png)

cat <<'EOF' > README.md
# Sprint 2 – Ubuntu Desktop Client + Samba Active Directory Integration

## Lab Activity – Domain Client, Users, Groups and GPO

---

## 1. Create Ubuntu Desktop Virtual Machine

Create an Ubuntu Desktop client virtual machine.

### Change Hostname

sudo hostnamectl set-hostname cli-ssd

---

## 2. Network Configuration

Configure IPv4 network settings as shown in the lab images.

### Connectivity Tests

Client:

ping 192.168.10.37
ping lab04.lan

Server:

ping cli-ssd

---

## 3. Hosts File Configuration

sudo nano /etc/hosts

Add server IP and domain name.

---

## 4. Netplan Configuration

Switch to root:

sudo su

Edit netplan:

nano /etc/netplan/01-network-manager-all.yaml

Apply:

sudo netplan apply

Fix permissions if needed:

sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
sudo chown root:root /etc/netplan/01-network-manager-all.yaml
sudo netplan apply

Verify routing:

ip route

---

## 5. Internet Access Troubleshooting

### Enable IP Forwarding (Server)

sudo nano /etc/sysctl.conf

Add at the end:

net.ipv4.ip_forward=1

Apply:

sudo sysctl -p
sysctl net.ipv4.ip_forward

### Configure NAT (Server)

sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo iptables -t nat -L -n -v

Verify internet from client.

---

## 6. Time Synchronization (NTP)

### Server

sudo apt update
sudo apt install chrony -y
sudo nano /etc/chrony/chrony.conf

Restart:

sudo systemctl restart chrony
sudo ufw allow 123/udp

### Client

sudo nano /etc/systemd/timesyncd.conf
sudo systemctl restart systemd-timesyncd

Verify:

timedatectl show-timesync --all

Alternative:

sudo apt-get install ntpdate
sudo ntpdate lab04.lan
sudo ntpdate -q lab04.lan

---

## 7. Install Required Packages (Client)

sudo apt update
sudo apt-get install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind

If errors:

sudo systemctl stop unattended-upgrades
sudo kill -9 4363
sudo dpkg --configure -a
sudo apt-get install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind

---

## 8. Kerberos Authentication Test

kinit administrator@LAB04.LAN
klist

---

## 9. Kerberos Configuration

sudo nano /etc/krb5.conf

Add:

dns_lookup_realm = true
dns_lookup_kdc = true

---

## 10. Samba Configuration

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.initial
sudo nano /etc/samba/smb.conf

Restart services:

sudo systemctl restart smbd nmbd
sudo systemctl stop samba-ad-dc
sudo systemctl enable smbd nmbd

---

## 11. Join Ubuntu Desktop to Domain

sudo net ads join -U administrator

Server verification:

sudo samba-tool computer list

---

## 12. Domain Authentication

sudo nano /etc/nsswitch.conf
sudo systemctl restart winbind

List users and groups:

wbinfo -u
wbinfo -g

Verify administrator:

getent passwd | grep administrator
id administrator

---

## 13. PAM Configuration

sudo pam-auth-update

Select:
Create home directory on login

Edit:

sudo nano /etc/pam.d/common-account

Add:

session required pam_mkhomedir.so skel=/etc/skel umask=0022

---

## 14. Login with Domain User

Login as:

administrator@lab04.lan

Grant sudo:

sudo usermod -aG sudo administrator

---

## 15. User and Group Management (Server)

Create group:

sudo samba-tool group add IT_departaments --group-scope=Universal --group-type=Security

Create user:

sudo samba-tool user create alice

Add user to group:

sudo samba-tool group addmembers IT_admins alice

---

## 16. Organizational Units

Create OU:

sudo samba-tool ou create "OU=IT_departaments,DC=lab04,DC=lan"

Move users:

sudo samba-tool user move alice "OU=IT_departaments,DC=lab04,DC=lan"
sudo samba-tool user move bob "OU=Students,DC=lab04,DC=lan"
sudo samba-tool user move charlie "OU=HR_Department,DC=lab04,DC=lan"

Move groups:

sudo samba-tool group move IT_admins "OU=IT_departaments,DC=lab04,DC=lan"
sudo samba-tool group move Students "OU=Students,DC=lab04,DC=lan"
sudo samba-tool group move IT_departaments "OU=HR_Department,DC=lab04,DC=lan"

Verify:

sudo samba-tool ou list

---

## 17. Group Policy Object (GPO)

Create GPO:

sudo samba-tool gpo create "IT_Security_Policy" -U Administrator

Link GPO:

sudo samba-tool gpo setlink "OU=IT_departaments,DC=lab04,DC=lan" {6DCC05BC-2848-4F4E-89E4-44EBE1A4C823} -U Administrator

---

## 18. Security Policies

Minimum password length:

sudo samba-tool domain passwordsettings set --min-pwd-length=8

Account lockout after 3 attempts:

sudo samba-tool domain passwordsettings set --account-lockout-threshold=3

Lockout duration:

sudo samba-tool domain passwordsettings set --account-lockout-duration=5



