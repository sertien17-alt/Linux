
# Sprint 2

## Ubuntu Desktop Client + Samba Active Directory Integration

---

## 1. Create Ubuntu Desktop Virtual Machine

Create an Ubuntu Desktop client virtual machine according to the lab requirements.

### Change the Hostname

sudo hostnamectl set-hostname cli-ssd

---

## 2. Network Configuration

Configure the IPv4 network settings as shown in the lab screenshots.

### Connectivity Tests

From the client:

ping 192.168.10.37  
ping lab04.lan  

From the server:

ping cli-ssd  

---

## 3. Hosts File Configuration

Edit the hosts file:

sudo nano /etc/hosts  

Add the server IP address and domain name.

---

## 4. Netplan Configuration

Switch to root user:

sudo su  

Edit the netplan configuration file:

nano /etc/netplan/01-network-manager-all.yaml  

Apply the configuration:

sudo netplan apply  

If permission errors appear, fix them:

sudo chmod 600 /etc/netplan/01-network-manager-all.yaml  
sudo chown root:root /etc/netplan/01-network-manager-all.yaml  
sudo netplan apply  

Verify routing:

ip route  

---

## 5. Internet Connectivity Troubleshooting

### Enable IP Forwarding on the Server

Edit the sysctl configuration:

sudo nano /etc/sysctl.conf  

Add at the end:

net.ipv4.ip_forward=1  

Apply and verify:

sudo sysctl -p  
sysctl net.ipv4.ip_forward  

---

### Configure NAT on the Server

sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE  
sudo iptables -t nat -L -n -v  

Verify internet access from the client.

---

## 6. Time Synchronization (NTP)

### Server Configuration

sudo apt update  
sudo apt install chrony -y  
sudo nano /etc/chrony/chrony.conf  

Restart the service:

sudo systemctl restart chrony  
sudo ufw allow 123/udp  

---

### Client Configuration

sudo nano /etc/systemd/timesyncd.conf  
sudo systemctl restart systemd-timesyncd  

Verify synchronization:

timedatectl show-timesync --all  

Alternative verification:

sudo apt-get install ntpdate  
sudo ntpdate lab04.lan  
sudo ntpdate -q lab04.lan  

---

## 7. Install Required Packages on Ubuntu Desktop

sudo apt update  
sudo apt-get install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind  

If installation fails:

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

Edit the Kerberos configuration file:

sudo nano /etc/krb5.conf  

Add the following lines:

dns_lookup_realm = true  
dns_lookup_kdc = true  

---

## 10. Samba Configuration on Client

Backup the Samba configuration file:

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.initial  

Edit the new configuration file:

sudo nano /etc/samba/smb.conf  

Restart Samba services:

sudo systemctl restart smbd nmbd  

Stop unnecessary AD DC service on the client:

sudo systemctl stop samba-ad-dc  

Enable required services:

sudo systemctl enable smbd nmbd  

---

## 11. Join Ubuntu Desktop to Samba AD Domain

Join the domain:

sudo net ads join -U administrator  

Verify from the server:

sudo samba-tool computer list  

---

## 12. Configure Domain Authentication

Edit NSS configuration:

sudo nano /etc/nsswitch.conf  

Restart winbind:

sudo systemctl restart winbind  

List domain users and groups:

wbinfo -u  
wbinfo -g  

Verify administrator account:

getent passwd | grep administrator  
id administrator  

---

## 13. PAM Configuration for Domain Users

Run PAM configuration tool:

sudo pam-auth-update  

Enable:
Create home directory on login  

Edit PAM configuration file:

sudo nano /etc/pam.d/common-account  

Add at the end:

session required pam_mkhomedir.so skel=/etc/skel umask=0022  

---

## 14. Login Using a Domain Account

Log in graphically with:

administrator@lab04.lan  

Grant sudo privileges:

sudo usermod -aG sudo administrator  

---

## 15. User and Group Management (Server)

Create a security group:

sudo samba-tool group add IT_departaments --group-scope=Universal --group-type=Security  

Create a user:

sudo samba-tool user create alice  

Add the user to a group:

sudo samba-tool group addmembers IT_admins alice  

---

## 16. Organizational Units (OU)

Create an OU:

sudo samba-tool ou create "OU=IT_departaments,DC=lab04,DC=lan"  

Move users:

sudo samba-tool user move alice "OU=IT_departaments,DC=lab04,DC=lan"  
sudo samba-tool user move bob "OU=Students,DC=lab04,DC=lan"  
sudo samba-tool user move charlie "OU=HR_Department,DC=lab04,DC=lan"  

Move groups:

sudo samba-tool group move IT_admins "OU=IT_departaments,DC=lab04,DC=lan"  
sudo samba-tool group move Students "OU=Students,DC=lab04,DC=lan"  
sudo samba-tool group move IT_departaments "OU=HR_Department,DC=lab04,DC=lan"  

Verify OUs:

sudo samba-tool ou list  

---

## 17. Group Policy Object (GPO)

Create a GPO:

sudo samba-tool gpo create "IT_Security_Policy" -U Administrator  

Link the GPO to the IT OU:

sudo samba-tool gpo setlink "OU=IT_departaments,DC=lab04,DC=lan" {6DCC05BC-2848-4F4E-89E4-44EBE1A4C823} -U Administrator  

---

## 18. Domain Security Policies

Set minimum password length:

sudo samba-tool domain passwordsettings set --min-pwd-length=8  

Set account lockout threshold:

sudo samba-tool domain passwordsettings set --account-lockout-threshold=3  

Set lockout duration (minutes):

sudo samba-tool domain passwordsettings set --account-lockout-duration=5  

---

## Result

The Ubuntu Desktop client was successfully joined to the Samba Active Directory domain and configured with:

- Kerberos authentication  
- Domain users and groups  
- Organizational Units (OU)  
- Group Policy Objects (GPO)  
- Security and password policies  

The system is ready for enterprise domain usage.
EOF
