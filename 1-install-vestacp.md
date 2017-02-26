## Install VestaCP Stock

1. Remove packages we don't need.  
```
apt-get purge apache* samba* bind9* exim4* postfix* -y  
apt-get autoremove --purge -y
```
2. Install curl
```
apt-get install curl
```
3. Get the VestaCP Installer
```
curl -O http://vestacp.com/pub/vst-install.sh
```
4. Run following bash command to install Nginx / PHP-FPM / MariaDB.  
   Update below command if you need Exim / FTP / Firewall etc. Generate new commands here: https://vestacp.com/#install
```
bash vst-install.sh --nginx yes --phpfpm yes --apache no --vsftpd no --proftpd no --exim no --dovecot no --spamassassin no --clamav no --named no --iptables no --fail2ban no --mysql yes --postgresql no --remi no --quota no --hostname YourHostName --email You@YourEmail.com --password YourPassword
```
5. Follow the setup instructions.
