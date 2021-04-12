# **Description**

- description needed

## **Preparation**

First, ensure that all the installed packages are entirely up to date

```Shell Scripting
apt update && \
  apt upgrade -y
```

### **Static IP**

Use the code below to edit the network file.

```Shell Scripting
sudo nano /etc/netplan/50-cloud-init.yaml
```

I am setting the raspberry pi to use wifi for now. Copy the text below into the .yaml file.

```Shell Scripting
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            optional: true
            access-points:
                "SSID":
                    password: "password"
            dhcp4: false
            addresses: [192.168.1.x/24]
            gateway4: 192.168.1.1
            nameservers:
              addresses: [8.8.8.8,8.8.4.4,192.168.1.1]
```

Then restart the network service.

```Shell Scripting
sudo netplan apply
```

```Shell Scripting
sudo wlan0 set power_save off
```

### **NginX Installation**

```Shell Scripting
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
```

### **Open Nginx Ports on UFW Firewall**

```Shell Scripting
sudo ufw enable
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 22
sudo ufw reload
sudo ufw status
```

### **Setup Auto Dynamic DNS Updater**

create a file called "google-domains-dynamic-dns-update.sh" at "/home/ubuntu"

```Shell Scripting
cd /home/ubuntu
sudo nano google-domains-dynamic-dns-update.sh
```

In the text editor, enter the following script and save the file. The username and password will be given after setting up the dynamic DNS.

```Shell Scripting
#!/bin/bash

### Google Domains provides an API to update a DNS
### "Synthetic record". This script updates a record with
### the script-runner's public IP address, as resolved using a DNS
### lookup.
###
### Google Dynamic DNS: https://support.google.com/domains/answer/6147083
### Synthetic Records: https://support.google.com/domains/answer/6069273

USERNAME="username"
PASSWORD="password"
HOSTNAME="hostname"

# Resolve current public IP
IP=$( dig +short myip.opendns.com @resolver1.opendns.com )
# Update Google DNS Record
URL="https://${USERNAME}:${PASSWORD}@domains.google.com/nic/update?hostname=${HOSTNAME}&myip=${IP}"
curl -s $URL

```

run it with the following inputs:

```Shell Scripting
sudo chmod +x google-domains-dynamic-dns-update.sh
./google-domains-dynamic-dns-update.sh
```

If it worked correctly, you should see the following:

> good x.x.x.x

Now we will set up a cron to run that script periodically so your dynamic hostname is always kept up to date.

```Shell Scripting
EDITOR=nano crontab -e
```

Then add the following to the crontab:

```Shell Scripting
0 * * * * /home/ubuntu/google-domains-dynamic-dns-update.sh
```

### **Obtain a free SSL Cert from Let's Encrypt**

For this we will need to have the latest version of 'snapd' installed. use the following command to do so:

```Shell Scripting
sudo snap install core; sudo snap refresh core
```

Next we will need to remove any old versions of Certbot and then re-install it. Use the following commands to do so:

```Shell Scripting
sudo apt-get remove certbot
sudo snap install --classic certbot
```

Now we can use the following command to ensure that the certbot command can be run:

```Shell Scripting
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### **Create site**

enter the following to create a site

```Shell Scripting
cd /etc/nginx/sites-available/
sudo nano www.example.com
```

In the text editor, input the following:

```Shell Scripting
server {
        listen 80;
        server_name cloud.example.com;
        root /home/ubuntu/www;
        index index.html;
}
```

### **Create a symbolic link for the configuration files we want to enable.**

```Shell Scripting
sudo ln -s /etc/nginx/sites-available/cloud.example.com /etc/nginx/sites-enabled/
```

### **Create the folder that NginX will server**

```Shell Scripting
cd /home/ubuntu
mkdir www
cd www
sudo nano index.html
sudo systemctl restart nginx
```

### **Give the site a cert**

```Shell Scripting
sudo certbot --nginx
```

choose your site and verify certbot gives you a cert.

### **Then restart the Nginx server**

```Shell Scripting
sudo systemctl restart nginx
```
