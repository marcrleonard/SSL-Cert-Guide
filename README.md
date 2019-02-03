# Certbot for NGINX

There are PLENTY of guides on how to do this... but sometimes it's just better to write your own to reference :-) 

#### Resources:
(Ubuntu 18 + Nginx): https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx



## Sample NGINX config
First, I rm'd the `default` config symlink.
Then made a config: `/etc/nginx/sites-enabled/testconfig`

```
server {
  root /var/www/yoursite;

  # note sure if this is right...
  listen 45542;
  listen 80;
  
  server_name whatever.com;

  access_log /var/log/nginx/pxs_access.log;
  error_log  /var/log/nginx/pxs_error.log;

  location / {
    root /var/www/yoursite;
  }
}
```

## Setup the correct certbot PPAs

Bash commands:
``` bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot python-certbot-nginx
```

## Check UFW

It's likely your cloud machine has UFW. Check to make sure your port 80 is open... This is sort of a temp catch 22 of the operation. It has to call out on port 80 to get https set up. Once you have the cert, you can close port 80.

### Check UFW rules:
`sudo ufw enable`
`sudo ufw status`
`sudo ufw app list`
Crazy enough, for my machine, `ssh` was NOT enabled by default, even though OpenSSH was in the 'app' list. So enable that, and `http` and `https`
`sudo ufw allow ssh`
`sudo ufw allow http`
`sudo ufw allow https`

## Set your A/DNS record properly
In digital ocean, I had to make sure my domain record was set to the ip addr of my droplet. When adding the entry you have to use an `@` to use the root of the domain. It should look like this at the end:

| Type   |      Hostname      |  Value |
|----------|:-------------:|------:|
| A |  whatever.com | directs to `ip.of.droplet` |


## Get your cert
`sudo certbot --nginx`
This will also edit your NGINX config file.  
It will ask you if you'd like to redirect http->https ... 
This is advised, but it will hose your site if you don't re-up your cert. 

## Read the cert output message carefully...
```
Congratulations! You have successfully enabled https://whatever.com
You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=whatever.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

**IMPORTANT NOTES:**
- Congratulations! Your certificate and chain have been saved at:
/etc/letsencrypt/live/whatever.com/fullchain.pem
Your key file has been saved at:
/etc/letsencrypt/live/whatever.com/privkey.pem
Your cert will expire on 2019-05-04. To obtain a new or tweaked
version of this certificate in the future, simply run certbot again
with the "certonly" option. To non-interactively renew *all* of
your certificates, run "certbot renew"
- If you like Certbot, please consider supporting our work by:

Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
Donating to EFF:  https://eff.org/donate-le
```
So, this is a 90 day cert. I read a few places online that people recommend renewing your certs every day... But I'm not sure it will let me. I will run once a month set that up with their suggested 'non-interactive' command.

## Setup a cron for your cert

`crontab -e`
`0 0 1 * *  certbot renew`



# Hopefully that's it!
