# **Raspberry Pi as a Home Network Firewall**
## Step 1 SSH (Secure Shell) into your Raspberry Pi 
  ### *If you have a spare monitor, keyboard, and mouse and are unexperienced with Secure Shell, feel free to set that up as an interface instead*

  #### Using Raspberry Pi imager and you SD card flash device, flash your SD with Raspberry Pi OS (64-bit), ensure to enable Wireless LAN if you plan to not have your Raspberry Pi connected to your Router via ethernet as well as under services, enable SSH using password authentication 

  #### Once your SD card has your OS flashed on to it, plug it into your Raspberry Pi 

  #### You can then use any open source software like RealVNC Viewer to SSH into your Raspberry Pi
---

## Step 2 Ensure your system is up to date

#### Open command prompt in the top left

#### Update and upgrade your Pi by entering the following code
> sudo apt update && sudo apt upgrade -y
---

## Step 3 Set up a DNS-Sinkhole using Pi-hole
### **A DNS-Sinkhole are very useful for routing malicious sources away from your host device There are many ways to configure these but for the point of this demonstration, we will be going over the basic configuration and the choice as for what it filters is up to you**

#### Ensure that curl is installed 
> sudo apt-get install curl

#### Install Pi-hole 
> curl -sSL https://install.pi-hole.net | bash

#### This will guide you through a few options including, if you would like to use a static IP (yes), choosing your DNS provider (any choice works but you can also research them), choosing your block lists (there is a menagerie of choices, I recommend applying as many as you like to include services that block advertisements as well as malicious services), as well as setting up your web interface

#### After installation, you'll be given an IP address and a password for your Pi-hole dashboard, enter that into your browser followed by /admin 

#### Enter your password and login

#### Before your Pi-Hole can start filtering your network's traffic, you first have to configure your network to use the IP address of your Raspberry Pi as your DNS server. You can access your network settings by entering your routers IP address or default gateway into your browser. Accessing your network settings can also be helpful for establishing or ensuring that your Raspberry Pi is using a static IP address. If you are using a EERO router device or other similar device that disables accessing networking setting through your browser, you can use the app to accomplish the same results

#### Your Pi-Hole should now be filtering! Feel free to further configure your block list and whitelists
---

## Step 4 Setting up iptables
### **This is important for blocking common ports that others can use against you in an attack to compromise your network**

#### Install iptables
> sudo apt-get install iptables

#### Block SOCKS Proxy (port 1080)
> sudo iptables -A INPUT -p tcp --dport 1080 -j DROP

#### Block Sun AnswerBook/Web server (port 8888)
> sudo iptables -A INPUT -p tcp --dport 8888 -j DROP

#### Block Nessus (port 3001)
> sudo iptables -A INPUT -p tcp --dport 3001 -j DROP

#### Block SCP-config (port 10001)
> sudo iptables -A INPUT -p tcp --dport 10001 -j DROP

#### Block Dynamic/Private Ports (port 49152)
> sudo iptables -A INPUT -p tcp --dport 49152 -j DROP

#### Install iptables-persistent to save your current rules  
> sudo apt-get install iptables-persistent

#### Save current rules
> sudo sh -c "iptables-save > /etc/iptables/rules.v4"
---

## Step 5 Enable IP Forwarding and Disable ICMP redirects
### **To make your Raspberry Pi a true firewall, you need to enable IP forwarding, this allows your Pi to route traffic between your network and the internet. Disabling ICMP redirects prevents MITM attacks**

#### Enable IP forwarding
> sudo sysctl -w net.ipv4.ip_forward=1

> sudo sysctl -w net.ipv6.ip_forward=1

#### Make the change permanent
> sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf

> sudo sed -i 's/#net.ipv6.ip_forward=1/net.ipv6.ip_forward=1/' /etc/sysctl.conf

#### Disable ICMP redirects
> sudo sysctl -w net.ipv4.conf.all.accept_redirects=0

> sudo sysctl -w net.ipv4.conf.all.send_redirects=0

> sudo sysctl -w net.ipv6.conf.all.accept_redirects=0

> sudo sysctl -w net.ipv6.conf.all.send_redirects=0

#### Make the change permanent
> sudo sed -i 's/#net.ipv4.conf.all.accept_redirects=0/net.ipv4.conf.all.accept_redirects=0/' /etc/sysctl.conf

> sudo sed -i 's/#net.ipv4.conf.all.send_redirects=0/net.ipv4.conf.all.send_redirects=0/' /etc/sysctl.conf

> sudo sed -i 's/#net.ipv6.conf.all.accept_redirects=0/net.ipv6.conf.all.accept_redirects=0/' /etc/sysctl.conf

> sudo sed -i 's/#net.ipv6.conf.all.send_redirects=0/net.ipv6.conf.all.send_redirects=0/' /etc/sysctl.conf
---

## Step 6 Configure NAT (Network Address Translation)
### **NAT allows your Raspberry Pi to act as a firewall by routing traffic between your internal network and the internet**

#### Post routing through Iptables 
> sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

> sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

#### Save the rules to ensure they persist after a reboot
> sudo apt-get install iptables-persistent

> sudo sh -c "iptables-save > /etc/iptables/rules.v4"

#### Apply the changes
> sudo sysctl -p
---

## And that's it, enjoy your open-source firewall!
Please feel free to leave recommendations, requests, fixes or complaints if anything is wrong. I will get fix anything reported and get back to you as soon as possible!
