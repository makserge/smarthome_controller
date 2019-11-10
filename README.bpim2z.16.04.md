# Smarthome controller

Smarthome project based on Ubuntu 16.04 and BananaPi M2 Zero SBC

Setup

1. Download bpi-m2z-8GB-sd.img.7z from

https://github-production-release-asset-2e65be.s3.amazonaws.com/112836752/da740980-aa64-11e9-8d6c-ed0c346736e0?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20191101%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20191101T001337Z&X-Amz-Expires=300&X-Amz-Signature=bf6212f0b53e210340b8daf3cbab7129b48ec42403f915aaeaee04fa9df402df&X-Amz-SignedHeaders=host&actor_id=16000579&response-content-disposition=attachment%3B%20filename%3Dbpi-m2z-8GB-sd.img.7z&response-content-type=application%2Foctet-stream

2. Flash using Etcher https://etcher.io or

Rufus http://rufus.akeo.ie/downloads/rufus-2.11p.exe

3. Boot and login to serial console 

ubuntu / ubuntu

4. Setup Wi-Fi network

sudo jed /etc/network/interfaces

update your network info like:


iface wlan0 inet dhcp
wpa-ssid "your AP"
wpa-psk "your ASCII passwd like 1234567890"

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

apt install -y nodejs


11. Install Mosquitto

apt install mosquitto

mosquitto_passwd -c /etc/mosquitto/passwd smarthome
Password:

nano /etc/mosquitto/conf.d/default.conf
add

allow_anonymous false
password_file /etc/mosquitto/passwd


systemctl restart mosquitto

12. Install Node-red

adduser node-red

Adding user `node-red' ...
Adding new group `node-red' (1002) ...
Adding new user `node-red' (1002) with group `node-red' ...
Creating home directory `/home/node-red' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for node-red
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y

usermod -aG sudo node-red


As node-red user
sudo npm install -g --unsafe-perm node-red node-red-admin

sudo npm install -g pm2
sudo pm2 start /usr/bin/node-red
sudo pm2 save
sudo pm2 startup

node-red-admin hash-pw
Password:

sudo chown -R node-red.node-red .node-red

nano /home/node-red/.node-red/settings.js

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

apt install ufw
ufw allow 1880
shutdown -r now

13. Open Node-Red dashboard
http://192.168.43.253:1880

and enter login / password

14. Connect CC2530 board to SBC

Pin  CC2530  SBC

+5V  +5V       +5V
GND  GND       GND
TX   P02       13
RX   P03       11


15. Setup UART2, UART3

cd /boot

dtc -I dtb -O dts -o m2z.dts bpi-m2-zero-v4.dtb_4.20.17

nano m2z.dts

replace

                serial@1c28800 {
                        compatible = "snps,dw-apb-uart";
                        reg = <0x1c28800 0x400>;
                        interrupts = <0x0 0x2 0x4>;
                        reg-shift = <0x2>;
                        reg-io-width = <0x4>;
                        clocks = <0x3 0x40>;
                        resets = <0x3 0x33>;
                        dmas = <0x15 0x8 0x15 0x8>;
                        dma-names = "rx", "tx";
                        status = "disabled";
                        pinctrl-names = "default";
                        pinctrl-0 = <0x20>;
                };

                serial@1c28c00 {
                        compatible = "snps,dw-apb-uart";
                        reg = <0x1c28c00 0x400>;
                        interrupts = <0x0 0x3 0x4>;
                        reg-shift = <0x2>;
                        reg-io-width = <0x4>;
                        clocks = <0x3 0x41>;
                        resets = <0x3 0x34>;
                        dmas = <0x15 0x9 0x15 0x9>;
                        dma-names = "rx", "tx";
                        status = "disabled";
                        pinctrl-names = "default";
                        pinctrl-0 = <0x21>;
                };

to

                serial@1c28800 {
                        compatible = "snps,dw-apb-uart";
                        reg = <0x1c28800 0x400>;
                        interrupts = <0x0 0x2 0x4>;
                        reg-shift = <0x2>;
                        reg-io-width = <0x4>;
                        clocks = <0x3 0x40>;
                        resets = <0x3 0x33>;
                        dmas = <0x15 0x8 0x15 0x8>;
                        dma-names = "rx", "tx";
                        status = "okay";
                        pinctrl-names = "default";
                        pinctrl-0 = <0x20>;
                };

                serial@1c28c00 {
                        compatible = "snps,dw-apb-uart";
                        reg = <0x1c28c00 0x400>;
                        interrupts = <0x0 0x3 0x4>;
                        reg-shift = <0x2>;
                        reg-io-width = <0x4>;
                        clocks = <0x3 0x41>;
                        resets = <0x3 0x34>;
                        dmas = <0x15 0x9 0x15 0x9>;
                        dma-names = "rx", "tx";
                        status = "okay";
                        pinctrl-names = "default";
                        pinctrl-0 = <0x21>;
                };



dtc -I dts -O dtb -o bpi-m2-zero-v4.dtb_4.20.17 m2z.dts

nano /etc/rc.local

add 

stty -F /dev/ttyS2 115200
chmod 666 /dev/ttyS2
stty -F /dev/ttyS3 115200
chmod 666 /dev/ttyS3

before

exit 0

 
shutdown -r now

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

19. Flash CC2530 module
https://www.aliexpress.com/item/ZigBee-Wireless-Module-CC2530-Module-Internet-Of-Things-Core-Board/32503616246.html

Reference
https://www.aliexpress.com/item/ZigBee-Wireless-Module-CC2530-Module-Internet-Of-Things-Core-Board/32503616246.html

Flashing proccess:

19.1 Download and unpack the archive with the library https://github.com/kirovilya/CCLib.

19.2 Update pins in Arduino/CCLib/Examples/CCLib_proxy/CCLib_proxy.ino

Replace

int CC_RST   = 5;
int CC_DC    = 4;
int CC_DD_I  = 3;
int CC_DD_O  = 2;

to

int CC_RST   = 2;
int CC_DC    = 4;
int CC_DD_I  = 5;
int CC_DD_O  = 16;


19.3 Flash NodeMCU Arduino/CCLib/Examples/CCLib_proxy/CCLib_proxy.ino sketch via Arduino IDE 

19.4 Connect CC2530 to NodeMcu

NodeMCU		CC2530 debug port

2		RESET
4		DC 
5		DD
16		DD
GND		GND
3.3V	3.3V
Vin		5V

19.5 Install Python 2.7 or later (tested with python 2.7.10), if not installed

Install pyserial 3.0.1, if not installed 
pip install pyserial==3.0.1

19.6 Set baudrate for serial port /dev/cu.SLAB_USBtoUART to 9600

19.7 Get information about the chip

CCLib-master/Python/cc_info.py -p /dev/cu.SLAB_USBtoUART

INFO: Found a CC2530 chip on /dev/cu.SLAB_USBtoUART

Chip information:
      Chip ID : 0xa524
   Flash size : 256 Kb
    Page size : 2 Kb
    SRAM size : 8 Kb
          USB : No

Device information:
 IEEE Address : 00124b0013de
           PC : 0000

Debug status:
 [ ] CHIP_ERASE_BUSY
 [ ] PCON_IDLE
 [X] CPU_HALTED
 [ ] PM_ACTIVE
 [ ] HALT_STATUS
 [ ] DEBUG_LOCKED
 [X] OSCILLATOR_STABLE
 [ ] STACK_OVERFLOW

Debug config:
 [ ] SOFT_POWER_MODE
 [ ] TIMERS_OFF
 [X] DMA_PAUSE
 [X] TIMER_SUSPEND

19.8 Download firmware 

wget https://github.com/Koenkk/Z-Stack-firmware/blob/master/coordinator/Z-Stack_Home_1.2/bin/default/CC2530_DEFAULT_20190608.zip?raw=true
19.9 Make firmware compatible with CCLib

Remove line before last one and save file as CC2530ZNP-Prod_mod_1.2.hex

19.10 Flash firmware (it takes about 1.5 hours)

python cc_write_flash.py -e -p COM4 --in=CC2530ZNP-Prod_mod_1.2.hex
python cc_write_flash.py -e -p COM4 --in=CC2530ZNP-with-SBL_mod.hex

python cc_write_flash.py -e -p /dev/cu.SLAB_USBtoUART --in=CC2530ZNP-Prod_mod.hex

INFO: Found a CC2530 chip on /dev/cu.SLAB_USBtoUART

Chip information:
      Chip ID : 0xa524
   Flash size : 256 Kb
    Page size : 2 Kb
    SRAM size : 8 Kb
          USB : No
Sections in CC2530ZNP-Prod_mod_1.2.hex:

 Addr.    Size
-------- -------------
 0x0000   8176 B
 0x1ff6   10 B
 0x3fff0   1 B
 0x2000   239616 B

This is going to ERASE and REPROGRAM the chip. Are you sure? <y/N>: y

Flashing:
 - Chip erase...
 - Flashing 4 memory blocks...
 -> 0x0000 : 8176 bytes
    Progress 100%... OK
 -> 0x1ff6 : 10 bytes
    Progress 100%... OK
 -> 0x3fff0 : 1 bytes
    Progress 100%... OK
 -> 0x2000 : 239616 bytes
    Progress 100%... OK

Completed

20. Add user zigbee

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

 
21. Install zigbee2mqtt 1.3

apt install -y git make g++ gcc python-dev

git clone https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt

cd /home/zigbee/zigbee2mqtt

git checkout 00ebd44
npm install

sudo chown -R zigbee:zigbee /home/zigbee/zigbee2mqtt

22. Configure zigbee2mqtt


mv /home/zigbee/zigbee2mqtt/data/configuration.yaml /home/zigbee/zigbee2mqtt/data/configuration.yaml.old

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

advanced:
  log_level: debug
  # Optional: Baudrate for serial port
  baudrate: 115200
  # Optional: RTS / CTS Hardware Flow Control for serial port
  rtscts: false
  
  
  # Home Assistant integration (MQTT discovery)
homeassistant: false

# allow new devices to join
permit_join: true

# MQTT settings
mqtt:
  # MQTT base topic for zigbee2mqtt MQTT messages
  base_topic: zigbee2mqtt
  # MQTT server URL
  server: 'mqtt://localhost'
  # MQTT server authentication, uncomment if required:
  # user: my_user
  # password: my_password
  user: 'smarthome'
  password: 'smarthome'

# Serial settings
serial:
  # Location of CC2531 USB sniffer
#  port: /dev/ttyACM0
  port: /dev/ttyS2

advanced:
  log_level: debug
  # Optional: Baudrate for serial port
  baudrate: 115200
  # Optional: RTS / CTS Hardware Flow Control for serial port
  rtscts: false


23. Start zigbee2mqtt

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

24. Setup zigbee2mqtt as a daemon as root

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

25. 
For the current CPU speed one can dynamically watch this change in real time using:

sudo watch -n 1  cat /sys/devices/system/cpu/cpu*/cpufreq/cpuinfo_cur_freq

and temperature

cat /sys/devices/virtual/thermal/thermal_zone0/temp