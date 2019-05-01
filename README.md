# How to make a reverse proxy
This little tutorial will help you (I hope) to create a reverse https proxy to redirect your incomming trafic from internet to a specific service (like on your NAS, access to Plex and your DSM with the same IP)

## Prerequisites
  - A Raspeberry Pi (or any computer you have with Ubuntu 18.04 LTS. Your gamming PC is way to powerfull, sorry...)
  - An internet connection (obviously)
  - ~ 30 mins
 
## Step 1 : Setup your ISP router
You need to redirect your incomming trafic to your reverse proxy server via NAT config (port fowarding). 

  - ***http*** :  `{INTERNET} ------------ 80 [ISP ROUTER] 80 ------------ [PROXY SERVER]` <br>
  - ***https*** : `{INTERNET} ------------ 443 [ISP ROUTER] 443 ------------ [PROXY SERVER]`
  
## Step 2 : Configure apache
  1. Install ***apache2*** on your server : <br>
      `$ sudo apt-get install apache2`. <br>
  2. Enable ***proxy***, ***proxy_http*** and ***ssl*** mods : <br>
      `$ sudo a2enmod proxy && sudo a2enmod proxy_http && sudo a2enmod ssl` <br>
  3. Restart ***apache2*** : <br>
      `$ sudo service apache2 restart` <br>
  4. Download and copy ***proxy_http.conf*** into the correct folder : <br>
      `$ sudo mv proxy_http.conf /etc/apache2/sites-available/` (**make sure you configured this file with your domain name and your destination server**) <br>
  5. Enable the site : <br>
      `$ sudo a2ensite proxy_http.conf` <br>
  6. Reload ***apache2*** : <br>
      `$ sudo service apache2 reload` <br><br>
!! If you try right now, **this doesn't work**. The http config redirect automaticly to the https config. So, we need to create a **https** config and sign your domain name with **Let's Encrypt** certificate !!

## Step 3 : Setup cerbot and sign your domain
  1. Install ***cerbot*** : <br>
    `$ sudo wget https://dl.eff.org/certbot-auto \` <br>
     `sudo mv certbot-auto /usr/local/bin/certbot-auto \` <br>
     `sudo chown root /usr/local/bin/certbot-auto \` <br>
     `sudo chmod 0755 /usr/local/bin/certbot-auto` <br>
  2. Launch ***cerbot*** configuration : <br>
    `$ sudo /usr/local/bin/certbot-auto --apache` <br><br>
    ***cerbot*** will scan your enabled sites and let you you choose which one sign
    ***cerbot*** generates a file (***proxy_http-le-ssl.conf***) on the `/etc/apache2/sites-available/` folder. Now, you need to compare ***proxy_https.conf*** from the repo and this file. Add any missing lines between these 2 configs (**especially all lines including SSL configuration**).<br><br>
  3. Enable your config : <br>
      `$ sudo a2ensite proxy_http-le-ssl.conf` <br><br>
**Et voilà !**

## Bonus step : Certbot automating renewal
With a ***cron*** tab, you can automaticaly renew your domain certificate <br>
  1. Setup ***cron*** :<br>
    `$ crontab -e`
  2. Copy this command in your cron file : <br>
    `0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot-auto renew `
