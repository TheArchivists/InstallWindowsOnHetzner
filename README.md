# InstallWindowsOnHetzner
How to install Windows server edition on a hetzner dedicated server

Hetzner has multiple ways of installing windows server editions on dedicated nodes, I'll try to cover some because neither hetzner nor anyone made any sensible fucking posting about this and its tiresome. 


List of methods 
- Install using Rescue system and VNC 
- Install using support ticket and KVM 
- Install using KVM and mount yourself 


I will take time and cover all 3 methods. 

Some stuff you need to know before you proceed because I wont re-re explain these 
Robot = hetzner web panel to manage servers
Rescue system = Is available under robot > servers > click on server you have > rescue (usually third option from the left)
KVM = A KVM switch is a hardware device that allows a user to control computers from one or more sets of keyboards, video monitors, and mice. Basically with kvm you can access the server straight from the BIOS, like you were actually there. For the nub nubs -  It's a bios level RDP to speak.
VNC viewer - Get the one for YOUR computer (not for your server) [from here](https://www.realvnc.com/en/connect/download/viewer/)

Here goes 

Method 1 
Installing Windows server image on hetzner using Rescue system and KVM

- Go to [this site](https://robot.your-server.de/server)
- Click server and from the list of servers if you have, click on your server 
- Click "rescue" and you will see this

![img1](https://i.imgur.com/Riqz6Nc.png)

- From here click "activate rescue system" 
- Now the page will give you ssh logins to rescue system, copy these somewhere safe we will need them later 
- Now go to "reset" section which is just to the left of "rescue" tab 

![img2](https://i.imgur.com/02uYdZY.png)

- Click "order an automatic hardware reset" - and then click send, this is like hard rebooting your server
- Now, when the server reboots it will reboot once into the rescue system mode

How will you know its up?
We need to ssh into the server and you will know if its up cause ssh will respond, install putty and connect to the IP of the server with port 22 and the logins that we saved earlier. 

Beep boop, you are in on ssh in rescue system. 

Let's get to real work from this point 
- Download and extract the portable qemu-kvm. /tmp folder is enough for this portable qemu-kvm. Just run this<br>
`wget -qO- /tmp https://github.com/AnimeKaizoku/InstallWindowsOnHetzner/raw/main/vkvm.tar.gz | tar xvz -C /tmp`

- If you dedicated server has a hard drive that is more than 2TB then you need to use the UEFI version too, run this one only if you have a drive that is bigger than 2tb in size (if you have multiple drives of 2tb you can ignore this step)<br>
`wget -qO- /tmp https://github.com/AnimeKaizoku/InstallWindowsOnHetzner/raw/main/uefi.tar.gz | tar -xvz -C /tmp`

- Okay, we are setup on our emulator, lets download a windows ISO and what better place than hetzner itself? 
Hetzner has 2 types of mirrors - Internal and external, the internal one is accessible without login from "inside the hetzner network" and external one is public. 
We will use the external one to see the path of the image we want and use the internal url (they both have same structure)

Visit [this site](http://download.hetzner.com/bootimages/)
The logins are
Username: hetzner
Password: download 

To save you the time I have the external url's here 

[Win Server STD CORE 2016 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2016_64Bit_English_-4_DC_STD_MLF_X21-70526.ISO)<br>
[Win Server STD CORE 2019 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO)<br>
[Win Server STD CORE 2022 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2022__64Bit_English_DC_STD_MLF_X22-74290.ISO)<br>

There are 3 images, I recommend server 2019 at this point of time - 2022 is too experimental.

Here is the internal URL: http://mirror.hetzner.de<br>
So just replace `download.hetzner.com` with `mirror.hetzner.de`

[Win Server STD CORE 2019 64Bit internal URL](http://mirror.hetzner.de/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO)

Now that's the URL that our server can access anytime without any logins needed. 

Now close your eyes and run these commands in serial order<br>
```
cd /tmp
wget http://mirror.hetzner.de/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO
cd /
```


- Now we run the qemu kun by running this command (Do not hit enter until you read the VNC section first you sick fucks)

For servers with no 2TB+ HDD
```
/tmp/qemu-system-x86_64 -net nic -net user,hostfwd=tcp::3389-:3389 -m 2048M -localtime -enable-kvm -cpu host,+nx -M pc -smp 2 -vga std -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO -hda /dev/sda -boot once=d -vnc :1
```

For servers with a 2TB+ HDD and UEFI
```
/tmp/qemu-system-x86_64 -bios /tmp/uefi.bin -net nic -net user,hostfwd=tcp::3389-:3389 -m 2048M -localtime -enable-kvm -cpu host,+nx -M pc -smp 2 -vga std -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO -hda /dev/sda -boot once=d -vnc :1
```

This is the VNC section
Once that is ready and before you hit enter on that ssh command lets open VNC viewer on your computer so we can connect to this RDP session that we just made 
Put the IP of the server in the connection field like this
yourIP:1

Where yourIP is ofc your server IP 
Example: 192.168.0.1:1


Now hit enter on the ssh, then hit connect on the VNC viewer
Hit connect, vnc will connect you to an rdp session where you will load the OS - in some cases, because its loading the OS like its from a pendrive you might get a "press and key to boot from disk/cd" so if you run the ssh command first you might miss this prompt and the ISO might not boot. 
This is why i said to prepare VNC first. 

From here its just a normal windows install, do that yada yada, install the os
Login to the server 

Go to my computer properties once your server is up > advanced system settings > remote > and tick the following 
![img3](https://i.imgur.com/BdmEbaL.png)

Save, then click reboot and then try to connect. 
Done
