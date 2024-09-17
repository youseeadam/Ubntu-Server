# UbntuServer
## Basic System Configuration
- Updating: sudo apt udpate, sudo apt upgrade
- Timezone: timedatectl list-timezones, sudo timedatectl set-timezone America/Los_Angeles
- NTP TIme:
  - sudo apt install ntp
  - sudo nano /etc/ntp.conf
    > pool.ntp.org<br>
    > time.nist.gov
  - sudo systemctl restart ntp
  - sudo ufw allow 123/udp
- Hostname:  sudo nano /etc/hostname
- IP Address: sudo nano /etc/netplan/50-cloud-init.yaml
- Installing webmin
  > curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh<br>
  > chmod +x setup-repos.sh<br>
  > sudo sh setup-repos.sh<br>
  > sudo  apt-get install --install-recommends webmin<br>
## Installing PXE Server
- sudo apt install tftpd-hpa
- sudo systemctl enable tftpd-hpa
- sudo nano /etc/default/tftpd-hpa
   > #/etc/default/tftpd-hpa<br>
   > TFTP_USERNAME="tftp"<br>
   > TFTP_DIRECTORY="/srv/tftp"<br>
   > TFTP_ADDRESS="0.0.0.0:69"<br>
   > TFTP_OPTIONS="--secure"<br>
- sudo chown -R nobody:nogroup /srv/tftp/
- sudo chmod -R 777 /srv/tftp/
- sudo systemctl restart tftpd-hpa
- test: sudo apt install tftp-hpa
- test: tftp 172.28.206.13
- In the folder /srv/tftp put the following files, these can be found in the windows toolkit and the Windows PE add-on
  - create a directory called Boot (case sensitive)
  - pxeboot.n12
  - boot.wim
  - other files....
## Installing DHCP Server
- sudo apt install isc-dhcp-server bind9 bind9utils
- cd /etc/bind
- sudo ddns-confgen FQDN_of_DNS_SERVER, see the next statement, you weel need part of that output
- sudo nano dhcp1.key and add the following lines from the above output
  > key "FQDN_of_DNS_SERVER" { <br>
  >          algorithm hmac-sha256;<br>
  >           secret "UwStCApuHV9XsaiTonPhVZYOZzXXDUuIgozobLwd/w0=";<br>
  > };
- Install webmn (just makes it easier)
  1. In webmn go to Servers --> Bind DNS Servers
 - create a Master Forward DNS zone for your forward lookup
   1. Domain: domain.com
   2. Add the master server (this server) in FQDN format
   3. check the box add NS record for master server
 - Create a Master Reverse Zone
   1. Zone: xxx.xxx.xxx (if using a /24 255.255.255.0).  Only use the octacts that are not being update
   2.  Add the master server (this server) in FQDN format
   3. check the box add NS record for master server
  - In webmn go to Servers --> DHCP Servers
   1. Create a new subnet.  This will match the reverse zone from DNS
   2. Just configure the Subnet Description, Network Address (typically xxx.xxx.xxx.0)
   3. Add the range and netmask
   4. Add a new DNS Zone
   5. For the IP of primary NS, it should be this server FQDN
   6. Name of the zone: fqdn (followed by . at the end): domain.com.
   7. Add another DNS Zone
   8. For the IP of primary NS, it should be this server FQDN
   9. in the name of the zone add the in-addr.arpa address: xxx.xxx.xxx.in-addr.arpa.  the x are in reverse order, so if the ip is 192.168.1.0 the name of the zone would be 1.158.192.in-addr.arp
   10. Configure everything else in shell
- SSH to the server
- cd: /etc/bind
- backup the named.conf.local: cp named.conf.local named.conf.local.bak
- sudo nano nano named.conf.local
- add to the top the following: (you created this file earlier)
  > include "/etc/bind/dhcp1.key";
- then in each zone you created add the following lines
  - for the forward zone
  > update-policy {<br>
  > grant FQDN_of_DNS_Server  wildcard *.Domain_Name A DHCID;<br>
  > };<br>
  - For the reverse Zone, note the in-addr-arpa should match the same as what you did above in DHCP
  > update-policy {<br>
  > grant FQDN_of_DNS_Server wildcard *.xxx.xxx.xxx.in-addr.arpa;<br>
  > };<br>
- cd /etc/dhcp
- sudo mkdir ddns-keys (this may already exist)
- sudo cp dhcpd.conf dhcpd.conf.old
- sudo nano ddns-keys/dhcp1.key: enter the exact same information used in the dhcp1.key file
- sudo nano dhcpd.conf add the following lines. there may already be sections that have some of this:
  > include "/etc/dhcp/ddns-keys/dhcp1.key";
  > ddns-updates on;
  > ddns-update-style standard;
- scroll down to the bottom where the subnets and zones are listed
- add the following to the subnet field: notice the quotes
  > ddns-domainname "domain.com";
- in each zone add the following.  The Key is from the dhcp1.key (notice the quotes are removed
  > key FQDN_of_DNS_SERVER;
- To add PXE boot add the following lines to either the zone or top of record
  > next-server xxx.xxx.xxx.xxx; <br>
  filename "\\Boot\\pxeboot.n.12";
### troubleshooting DNS and DHCP
- restarting dhcp server: sudo systemctl restart isc-dhcp-server
- Getting the status of DHCP server: sudo systemctl status isc-dhcp-server
- Starting bind: sudo systemctl start bind9
- Getting the status of bind: sudo systemctl status bind9
- Looking through the log files: sudo tail -f /var/log/syslog
- When editng dns entries in webmin, you must first freeze the zone
## Installing Samba
- Install webmn, it helps when dealing with files.  But not required
- fdisk -l to list all the disks
- to configure the disk: fdisk /dev/sdX
- press n to create a new partiion
- press w to write the configuration
- format the parition: mkfs -t ext4 -q /dev/sdx1
- create a directory to mount: sudo mkdir /srv/imaging
- mount: sudo mount -t ext4 /dev/sdx1 /srv/imaging
- Permently mount:
- sudo nano /etc/fstab
  >/dev/sdx1       /srv/imaging    ext4    defaults        0       0
 - create a new share: sudo nano /etc/samba/smb.conf
   > [imaging]<br>
   > comment = system deployment images<br>
   > path = /srv/imaging<br>
   > read only = no<br>
   > browsable = yes<br>
   > writeable = Yes<br>
- Fix perrmisions on the share: 
  - sudo chown nobody:nogroup /srv
  - sudo chmod 777 /srv/imaging/
