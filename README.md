# UbntuServer
## System Configuration
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
## Installing DHCP Server
