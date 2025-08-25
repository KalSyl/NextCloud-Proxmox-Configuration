## Description

Below are the steps that I followed to get a NextCloud LXC set up and running on my Proxmox server with Reverse Proxy and NAS storage. I have a NAS (TrueNas) already set up and configured running on a VM inside Proxmox (it's a bad idea, so I wouldn't do this if I had the hardware to dedicate to the NAS). Functionally, it *should* work the same way regardless of where your NAS is mounted. I also have nginx installed and working with cloudflare already. 

## Other Guides

I used [this guide for installing nginx](https://medium.com/@rar1871/nginx-installing-proxy-manager-in-lxc-v2-debian-d4d4c98109b1)\
I used [this guide for setting up cloudflare](https://silicon.blog/2023/01/22/how-to-combine-nginx-proxy-manager-with-cloudflare-to-access-your-websites-web-services-securely/)\
I used [this guide for setting up TrueNas](https://www.youtube.com/watch?v=MkK-9_-2oko)

If these links are dead, I wish you the best!!

## LXC Install

In Proxmox, click on your local stoarge in the left hand pane. Then click on CT Templates.
![Image 1](https://github.com/KalSyl/NextCloud-Proxmox-Configuration/blob/main/Tutorial_Pictures/Step_1.png?raw=true)

Click on "Templates" and search for "Nextcloud". Then select the most recent version and click "Download"
![Image 2](https://github.com/KalSyl/NextCloud-Proxmox-Configuration/blob/main/Tutorial_Pictures/Step_2.png?raw=true)

Once the task completes, close the window and create a new container using this template. Click on "Create CT", and set the name and password as desired. Click "Next".

On the template section, select the template we downloaded in the previous step. Click "Next".

Pick your disk size. I typically leave it at 8GB since we will be using a NAS share to store data instead of the local storage. Click "Next".

Select number of cores. For just file storage, 2 cores should be sufficient. If you want to install the office apps or other apps, 4 is better. Click "Next".

Select Memory. I usually do 4096 for a normal install, and 8192 when wanting to use office/apps. Click "Next".

Configure Nextwork. Static IP can be set to whatever you want. If you have no idea, close the window and click on the node, and then shell. Enter the following command:
```
ip route | grep default
```
This shows your gateway. Keep note of this. Repeat the above steps, and when you get to the network configuration, enter this into the gateway box, and set your IP address with the first 3 numbers, and whatever you'd like for the 4th. Somewhere on the internet someone told me to use the container IDs as the last number for organization, and I think that's a great idea! Remember to add '/24' at the end of your IP address (but not the gateway). 

Click Next, and then Finish. Let the installer do its thing and when it says "TASK OK", close it out and then click on the new container and click "Start".

As the container boots, it will install a few things, then ask you to log in. Use "root" as the login, and the password you set during creation. A blue GUI will open up so we can enter a few passwords. Keep track of these passwords, especially the admin password. 
![Image 3](https://github.com/KalSyl/NextCloud-Proxmox-Configuration/blob/main/Tutorial_Pictures/Step_3.png?raw=true)

At the Nextcloud Domain step, enter the IP you made during setup. This is important so we can set up the proxy. 
![Image 4](https://github.com/KalSyl/NextCloud-Proxmox-Configuration/blob/main/Tutorial_Pictures/Step_4.png?raw=true)

You can skip the API Key section and Email section, but install the updates. Let the terminal do its thing, and grab a snack. Once you see the "Nextcloud-Tutorial appliance services" box, NextCloud is installed and ready to go!

## Configure Nextcloud for Reverse Proxy

Now that our NextCloud is set up, let's get external access set up!

Log into your cloudflare account, and click on DNS in the left pane. Click on "Add Record". We want an "A" record, input the subdomain you want, and the IPv4 address of your Public IP (hopefully you have a static IP or your ISP doesn't update them often). Save the entry.

Navigate to the IP of your nginx proxy manager, and click on "Add Proxy Host". Add the subdomain you picked in Cloudflare, and the domain you set up in cloudflare. Should look like this:
```
nextcloud.yourdomain.com
```
Choose https, enter the IP address of the NextCloud container, and enter port 443, and select "Block Common Exploits".
Click on the SSL tab, and get an SSL certificate, and delect "Force SSL", "HTTP/2 Support", "HSTS Enabled", then save. 

Now we'll need to go into the nextcloud container to add our domain as trused so we can access it.

Navigate to the nextcloud console, and press Ctrl+C if the blue screen is still up. This will let us get into the terminal.

Now enter the following command:
```
nano /var/www/nextcloud/config/config.php
```
This will open the config file so we can edit it. Use arrow keys to navigate and add the following line.
```
2 => 'nextcloud.yourdomain.com',
```
![Image 5](https://github.com/KalSyl/NextCloud-Proxmox-Configuration/blob/main/Tutorial_Pictures/Step_5.png?raw=true)

Ctrl+X to exit, Y to save, and Enter to overwrite the old file. You should now be able to access your nextcloud server through your reverse proxy!

And that's all we need to do to get the reverse proxy set up!

## Configuring Next
