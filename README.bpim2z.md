# Smarthome controller

Smarthome project based on Ubuntu 19.10 and BananaPi M2 Zero SBC

Setup

1. Download bpi-m2z-8GB-sd-ubuntu_eoan_kernel_fbdev_5.3.11.img.7z from

https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v2.8/bpi-m2z-8GB-sd-ubuntu_eoan_kernel_fbdev_5.3.11.img.7z

2. Flash using Etcher https://etcher.io or

Rufus http://rufus.akeo.ie/downloads/rufus-2.11p.exe

3. Boot and login to serial console 

ubuntu / ubuntu

4. Setup static IP for LAN

sudo rfkill block 1

sudo jed /etc/netplan/01-netcfg.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      # Ser IP address & subnet mask
      addresses: [192.168.137.100/24]
      # Set default gateway
      gateway4: 192.168.137.1
      nameservers:
        # Set DNS name servers
        addresses: [192.168.137.1,8.8.8.8]
      dhcp6: no
  wifis:
    wlan0:
      dhcp4: yes
      access-points:
        "my-router-name":
          password: "<wifi password here in plain text>"	  
	  
sudo netplan apply
sudo rfkill block 1
  
sudo apt purge network-manager

sudo systemctl disable dhclient.service
sudo systemctl enable networking

sudo rfkill unblock 1
sudo reboot

5. Set new root password

sudo passwd root

New password:
Retype new password:
passwd: password updated successfully

su -

6. Update system

apt update
apt upgrade

7. Create new user

adduser smarthome

Adding user `smarthome' ...
Adding new group `smarthome' (1001) ...
Adding new user `smarthome' (1001) with group `smarthome' ...
Creating home directory `/home/smarthome' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for smarthome
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y

8. Add nano

apt install nano -y

9. Set timezone

timedatectl set-timezone Europe/Kiev

10. Install NodeJS

apt install curl -y

curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -

apt install -y nodejs npm
export NODE_TLS_REJECT_UNAUTHORIZED=0
npm config set strict-ssl false
npm install npm@latest -g

11. Install Mosquitto

apt install -y mosquitto

mosquitto_passwd -c /etc/mosquitto/passwd smarthome
Password:

nano /etc/mosquitto/conf.d/default.conf
add

allow_anonymous false
password_file /etc/mosquitto/passwd


systemctl restart mosquitto

12. Install Node-red

npm install -g --unsafe-perm node-red

npm install -g pm2
pm2 start /usr/local/bin/node-red
pm2 save
pm2 startup

npm install -g --unsafe-perm node-red-admin

node-red-admin hash-pw
Password:

nano /root/.node-red/settings.js

replace


//adminAuth: {
//    type: "credentials",
//    users: [{
//        username: "admin",
//        password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcx$
//        permissions: "*"
//    }]
//},


to 

adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "*",
        permissions: "*"
    }]
},


reboot

13. Open Node-Red dashboard
http://192.168.43.253:1880

and enter login / password

14. Connect CC2530 board to SBC

Pin  CC2530  SBC

+5V  +5V       +5V
GND  GND       GND
TX   P02       13
RX   P03       11


15. Setup UART2

apt install device-tree-compiler

cd /boot

dtc -I dtb -O dts -o m2z.dts bpi-m2-zero-v4.dtb_5.3.11-fbdev

nano m2z.dts

replace

serial@1c28800 {
                        compatible = "snps,dw-apb-uart";
                        reg = < 0x1c28800 0x400 >;
                        interrupts = < 0x00 0x02 0x04 >;
                        reg-shift = < 0x02 >;
                        reg-io-width = < 0x04 >;
                        clocks = < 0x03 0x40 >;
                        resets = < 0x03 0x33 >;
                        dmas = < 0x15 0x08 0x15 0x08 >;
                        dma-names = "rx\0tx";
                        status = "disabled";
                        pinctrl-names = "default";
                        pinctrl-0 = < 0x20 >;
                };


to

serial@1c28800 {
                        compatible = "snps,dw-apb-uart";
                        reg = < 0x1c28800 0x400 >;
                        interrupts = < 0x00 0x02 0x04 >;
                        reg-shift = < 0x02 >;
                        reg-io-width = < 0x04 >;
                        clocks = < 0x03 0x40 >;
                        resets = < 0x03 0x33 >;
                        dmas = < 0x15 0x08 0x15 0x08 >;
                        dma-names = "rx\0tx";
                        status = "okay";
                        pinctrl-names = "default";
                        pinctrl-0 = < 0x20 >;
                };


dtc -I dts -O dtb -o bpi-m2-zero-v4.dtb_5.3.11-fbdev m2z.dts


nano /etc/rc.local

#!/bin/bash
stty -F /dev/ttyS2 115200
chmod 666 /dev/ttyS2
stty -F /dev/ttyS3 115200
chmod 666 /dev/ttyS3

exit 0


chmod 755 /etc/rc.local

systemctl restart rc-local
systemctl daemon-reload

systemctl status rc-local

● rc-local.service - /etc/rc.local Compatibility
   Loaded: loaded (/lib/systemd/system/rc-local.service; enabled-runtime; vendor preset: enabled)
  Drop-In: /usr/lib/systemd/system/rc-local.service.d
           └─debian.conf
   Active: active (exited) since Sun 2019-12-01 00:04:03 EET; 2s ago
     Docs: man:systemd-rc-local-generator(8)
   CGroup: /system.slice/rc-local.service


 
reboot

16. Check UART2, UART3

stty -F /dev/ttyS2

speed 115200 baud; line = 0;
-brkint -imaxbel

stty -F /dev/ttyS3

speed 115200 baud; line = 0;
-brkint -imaxbel

17. Connect OLED to SBC

Pin		OLED	SBC

+3.3V	VCC		+3.3V
GND  	GND		GND
SCL		SCL		5
SDA		SDA		3

18. Install OLED status monitor

git clone https://github.com/zhaolei/WiringOP.git -b h3 
cd WiringOP
chmod +x ./build
sudo ./build
cd ..
ldconfig

git clone https://github.com/nopnop2002/ssd1306_rpi.git
cd ssd1306_rpi

cc -o oled oled.c fontx.c -lwiringPi -lcrypt -lm -lrt -lpthread -DI2C  
bash ./test.sh 

wget https://raw.githubusercontent.com/makserge/smarthome_controller/master/pushbuttons.c

gcc -lwiringPi -lcrypt -lm -lrt -lpthread -o pushbuttons pushbuttons.c
./pushbuttons

nano /usr/local/bin/run23release.sh

#!/bin/bash

cpuTemp=`cat /sys/devices/virtual/thermal/thermal_zone0/temp`
cpuTempDegree=$((cpuTemp/1000))

freeRAM=`free | grep Mem | awk '{print $4/$2 * 100.0}'`
freeRAMRounded=`printf "%.1f" $freeRAM`

usedSpace=`df --output=pcent / | awk -F'%' 'NR==2{print $1}'`
freeSpace=$((100-$usedSpace))

cd /root/ssd1306_rpi

./oled r
./oled +1 `hostname -I | cut -d' ' -f1` 
./oled +2 "Temp: $cpuTempDegree"
./oled +3 "Free RAM: $freeRAMRounded%"
./oled +4 "Free space: $freeSpace%"
./oled s
sleep 10

./oled r
./oled s


chmod +x /usr/local/bin/run23release.sh

nano /usr/local/bin/run23short.sh
#!/bin/bash

chmod +x /usr/local/bin/run23short.sh


nano /etc/systemd/system/pushbuttons.service

[Unit]
Description=pushbuttons
After=network.target

[Service]
ExecStart=/root/pushbuttons
WorkingDirectory=/root
StandardOutput=inherit
StandardError=inherit
Restart=always
User=root

[Install]
WantedBy=multi-user.target



sudo systemctl start pushbuttons
sudo systemctl enable pushbuttons.service

17. Add user zigbee

adduser zigbee

Adding user `zigbee' ...
Adding new group `zigbee' (1002) ...
Adding new user `zigbee' (1002) with group `zigbee' ...
Creating home directory `/home/zigbee' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for zigbee
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y

 
18. Install zigbee2mqtt

apt install -y git make g++ gcc python-dev

19. Login as zigbee user

git clone https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt

cd /home/zigbee/zigbee2mqtt

npm install

20. Configure zigbee2mqtt

nano /home/zigbee/zigbee2mqtt/data/configuration.yaml

Replace 

server: 'mqtt://localhost'
# MQTT server authentication, uncomment if required:
# user: my_user
# password: my_password

to 

server: 'mqtt://localhost'
user: '***'
password: '***'

Replace 

port: /dev/ttyACM0

to 

port: /dev/ttyS2

Add to the end

advanced:
  log_level: debug
  # Optional: Baudrate for serial port
  baudrate: 115200
  # Optional: RTS / CTS Hardware Flow Control for serial port
  rtscts: false


21. Start zigbee2mqtt

cd /home/zigbee/zigbee2mqtt
npm start

When started successfully, you will see:

  zigbee2mqtt:info 2018-11-17 18:13:34 Logging to directory: '/home/zigbee/zigbee2mqtt/data/log/2018-11-17.18-13-34'
  zigbee2mqtt:debug 2018-11-17 18:13:35 Using zigbee-shepherd with settings: '{"net":{"panId":6754,"channelList":[11]},"dbPath":"/home/zigbee/zigbee2mqtt/data/database.db","sp":{"baudRate":115200,"rtscts":false}}'
  zigbee2mqtt:debug 2018-11-17 18:13:35 Can't load state from file /home/zigbee/zigbee2mqtt/data/state.json (doesn't exsist)
  zigbee2mqtt:info 2018-11-17 18:13:35 Starting zigbee2mqtt version 0.2.0 (commit #b0d3c2f)
  zigbee2mqtt:info 2018-11-17 18:13:35 Starting zigbee-shepherd
  zigbee2mqtt:info 2018-11-17 18:13:36 zigbee-shepherd started
  zigbee2mqtt:info 2018-11-17 18:13:36 Coordinator firmware version: '20182308'
  zigbee2mqtt:debug 2018-11-17 18:13:36 zigbee-shepherd info: {"enabled":true,"net":{"state":"Coordinator","channel":11,"panId":"0x1a62","extPanId":"0xdddddddddddddddd","ieeeAddr":"0x00124b0013def197","nwkAddr":0},"firmware":{"transportrev":2,"product":0,"version":"2.6.3","revision":20182308},"startTime":1542478416,"joinTimeLeft":0}
  zigbee2mqtt:info 2018-11-17 18:13:36 Currently 0 devices are joined:
  zigbee2mqtt:warn 2018-11-17 18:13:36 `permit_join` set to  `true` in configuration.yaml.
  zigbee2mqtt:warn 2018-11-17 18:13:36 Allowing new devices to join.
  zigbee2mqtt:warn 2018-11-17 18:13:36 Set `permit_join` to `false` once you joined all devices.
  zigbee2mqtt:info 2018-11-17 18:13:36 Zigbee: allowing new devices to join.
  zigbee2mqtt:info 2018-11-17 18:13:36 Connecting to MQTT server at mqtt://localhost
  zigbee2mqtt:info 2018-11-17 18:13:36 zigbee-shepherd ready
  zigbee2mqtt:info 2018-11-17 18:13:36 Connected to MQTT server
  zigbee2mqtt:info 2018-11-17 18:13:36 MQTT publish, topic: 'zigbee2mqtt/bridge/state', payload: 'online'
  zigbee2mqtt:debug 2018-11-17 18:13:36 Soft reset timeout disabled

22. Setup zigbee2mqtt as a daemon as root

nano /etc/systemd/system/zigbee2mqtt.service

[Unit]
Description=zigbee2mqtt
After=network.target

[Service]
ExecStart=/usr/bin/npm start
WorkingDirectory=/home/zigbee/zigbee2mqtt
StandardOutput=inherit
StandardError=inherit
Restart=always
User=zigbee

[Install]
WantedBy=multi-user.target



Start daemon

sudo systemctl start zigbee2mqtt

Verify

systemctl status zigbee2mqtt.service

* zigbee2mqtt.service - zigbee2mqtt
   Loaded: loaded (/etc/systemd/system/zigbee2mqtt.service; disabled; vendor pre
   Active: active (running) since Tue 2019-10-22 02:36:37 EEST; 12s ago
 Main PID: 6877 (npm)
    Tasks: 23 (limit: 855)
   Memory: 35.0M
   CGroup: /system.slice/zigbee2mqtt.service
           |-6877 npm
           |-6889 sh -c node index.js
           `-6890 node index.js

Oct 22 02:36:37 nanopineo systemd[1]: Started zigbee2mqtt.

Enable

sudo systemctl enable zigbee2mqtt.service


Reference

# Stopping zigbee2mqtt
sudo systemctl stop zigbee2mqtt

# Starting zigbee2mqtt
sudo systemctl start zigbee2mqtt

# View the log of zigbee2mqtt
sudo journalctl -u zigbee2mqtt.service -f

