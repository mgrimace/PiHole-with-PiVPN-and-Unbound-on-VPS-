# PiHole with PiVPN and Unbound on VPS 
 These are my install notes for creating a virtual private server (VPS; Amazon AWS EC2 free tier) with PiHole, PiVPN (wireguard), and unbound on the VPS to connect to my clients (phone, laptop, family's devices, etc). I’m able to connect to PiVPN with my wireguard profile that I generated (e.g., for my phone), and access my PiHole admin page. Split-tunnelling is used to only route the DNS (vs., all data) for speed and to save bandwidth on the free tier.

## What is this?
* This setup lets you run [PiHole](https://github.com/pi-hole/pi-hole), from anywhere, for free without needing any hardware
* Basically, you'll be setting up PiHole on a virtual private server (VPS), connecting to your virtual PiHole using a VPN called PiVPN.
* This setup forces your devices to use only the DNS provided by the PiVPN connection (i.e., the PiHole; this is called a split-tunnel), not your full data (i.e., full tunnel). This makes it fast/light and keeps it free.
* This setup uses WireGuard to connect to your PiVPN (and PiHole) which makes it fairly easy to add to devices.
* I used a free tier of Amazon Web Services [AWS] but this should work on whatever ones you choose (e.g., Google, etc.)
* You can then keep connected to PiHole from any devices (e.g., laptop, phone, etc.), from anywhere (i.e., not just on your home network)
* It's relatively easy to do yourself, and since it's all done manually (vs., a script) you can learn a bit as you go!
* update: added unbound recursive DNS server for safety/privacy!

# Table of Contents
* [Create a VPS](#create-a-VPS)
* [Install PiHole](#Install-PiHole)
* [Install and setup VPN](#Install-and-setup-your-VPN)
* [Install Unbound](#Install-Unbound)
* [Notes and Troubleshooting](#Notes)

# Create a VPS
## Step 1: Create a free Ubuntu server in AWS


* Create an AWS account
* Go to EC2, click launch instance, select “free tier” and choose Ubuntu (I picked 20.04)
* Manually configure, and click through each step until you get to Security groups, and add the following: Custom UDP, Port Range: 51820, Source: 0.0.0.0/0, and Description: Wireguard
* Download your new keypair and save it to .ssh (on mac, I created a folder called .ssh in my main user folder). I called mine PiVPNHOLE
* In your EC2 terminal, note your PublicDNS (IPv4), it’ll look like: ec2-###.location.com, I call this [your host] below
* Click Elastic IP to create an Elastic IP, then click actions and associate, and associate the Elastic IP to your new server

# Install PiHole
## Step 2: Use Terminal to connect to your new Ubuntu server

```ssh -i /Users/[your user]/.ssh/PiVPNHOLE.pem ubuntu@[your host]``` 

## Step 3: install PiHole

```curl -sSL https://install.pi-hole.net | bash```
* Take note of your PiHole's web interface IP and the password

# Install and setup your VPN
## Step 4: install PiVPN
```curl -L https://install.pivpn.io | bash```

## Step 5: create your user profiles

```pivpn add -n [config name]```
*  where [config name] is a unique name for each of your devices (e.g., mphone, mlaptop). You can repeat this step for as many devices that you want to connect to your Pi-hole.

## Step 6: add split-tunnel connection to your config
```sudo nano /etc/wireguard/configs/[config name].conf```
* Change AllowedIPs from "0.0.0.0/0, ::0" to "[PiHole IP address]/32, [DNS IP]/32". 
* The [DNS IP] is listed in [interface] and by default 10.6.0.1/32.
* Note spit-tunnelling only routes the DNS (i.e., PiHole ad-blocking) vs., all of your data through your VPN which will save bandwidth to keep you on the free tier.
* You'll need to repeat this for each [config name] you created in step 5.

## Step 7: display your config QR code to connect your mobile device
```pivpn -qr [config name, e.g., mphone]```

## Step 8: download your configs for laptops/desktops

```scp -i /Users/[Your Name]/.ssh/PiVPNHOLE.pem ubuntu@[your host]:~/configs/[config name, e.g., mlaptop] [destination on your computer, e.g., ~/Documents/wireguard]```

Here's an example because this can be confusing:

```scp -i /Users/example/.ssh/PiVPNHOLE.pem ubuntu@ec2-1-2-3-4.location.compute.amazonaws.com:~/configs/ /Users/example/wireguard```

This will download all of your config files to a folder on your computer called wireguard

## Step 9: Install Wireguard on your device

* Scan your QR code for your mobile devices, and/or install the downloaded configs for your laptop/desktop/other devices, turn them on.
* I set them to "on-demand" meaning it'll always be on
* Go check out your PiHole at the address you saved in Step 2!

# Install Unbound
## Step 9: Install Unbound

* Connect back to your Ubuntu VPS in terminal
* Follow this [guide](https://docs.pi-hole.net/guides/unbound/), it's pretty straight forward
* One thing to note, when you get to Configure Unbound instruction, it'll say "/etc/unbound/unbound.conf.d/pi-hole.conf", you actually need to add "sudo nano" to the start of that code in your Terminal to be able to create and paste in the configuration (or else you'll just get an error). Then just copy/paste in the text from the guide and hit save/exit.
```sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf```

# Notes
## Troubleshooting
* Before being able to remotely log in, I had to run the command ```chmod 600 /Users/[your name]/.ssh/PiVPNHOLE.pem```
* After clicking "generate keys" in PiVpn, you may get `/tmp/setupVars.conf permission denied`. I solved this by deleting that file.
* You may need to run the piVpn script as sudo. Run with `curl -L https://install.pivpn.io |  sudo bash`

## Credits
* Thanks to @SuspectTyrannosaurus for fixing creating user profiles.
* Special thanks to @DasJason for inspiring this project, troubleshooting, and various code tips/tricks.
* Thanks to Thank you to @afruitpie for helping me figure out split-tunnelling and how to download the configuration files. 
* Thanks also to @kryten2k35 for making sure this method of PiHole isn't exposed to the entire world (i.e., double checking port 53 is closed so the DNS isn't public).

<h2>Buy me a coffee</h2>

If you found this guide helpful, please consider buying me a coffee by clicking the link below to donate a few dollars. I'll do my best to keep this guide up to date and as user-friendly as possible. Thank you and take good care

<a href="https://www.buymeacoffee.com/cammarata.m" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>
