# Smarthome controller

Smarthome project based on Ubuntu 16.04 and BananaPi M2 Zero SBC

Setup

1. Download bpi-m2z-8GB-sd.img.7z from

https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v15/bpi-m2z-8GB-sd.img.7z

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

apt install mosquitto -y

mosquitto_passwd -c /etc/mosquitto/passwd smarthome
Password:

nano /etc/mosquitto/conf.d/default.conf
add

allow_anonymous false
password_file /etc/mosquitto/passwd


systemctl restart mosquitto

12. Install Node-red

sudo npm install -g --unsafe-perm node-red node-red-admin 

sudo npm install -g pm2
sudo pm2 start /usr/bin/node-red
sudo pm2 save
sudo pm2 startup

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

apt install ufw -y
ufw allow 1880
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

apt install git -y
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
cd ..

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
./oled +2 "Temp: $cpuTempDegree C"
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


systemctl start pushbuttons
systemctl enable pushbuttons.service

19. Flash E18-MS1PA1-IPX Zigbee CC2530 2.4Ghz 100mW IPX Antenna IoT uhf Wireless Transceiver 2.4g Transmitter Receiver Module CC2530 PA
https://www.aliexpress.com/item/32640770110.html


Flashing proccess:

19.1 Download and unpack VLK DIY Multi Flasher (VLK_DIY_Multi_Flasher.rar) from

https://myzigbee.ru/books/%D0%BF%D1%80%D0%BE%D1%88%D0%B8%D0%B2%D0%BA%D0%B8/page/vlk-diy-multi-flasher

19.2 Connect ESP32 board to Windows PC and start VLK DIY Multi_Flasher.exe
19.3 Switch to ZESP tab
19.4 Select ESP's COM port
19.5 Press "Write full flash" and wait for 3 minutes for finish
19.6 Connect IO21 and GND on ESP32
19.6 Connect Module to ESP32

ESP32	CC2530 module

5		RESET 	24
18		DC 		3
19		DD		4
GND		GND		1
3.3V	3.3V	2

19.7 Press Connect and check if module returned:

ChipID: A5
Chip: CC2530
Rev: 24
Size: 256k

19.8 Download firmware from goofyk(https://github.com/kirovilya)

https://www.dropbox.com/s/vl6b5q7kaflrbgm/CC2530ZNP.hex?dl=0

19.9 Flash firmware:

Choose CC2530ZNP.hex and press "Write"
Wait for Wrote Flash message (about 70s)


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

 
21. Install zigbee2mqtt 1.7.1

apt install -y git make g++ gcc python-dev

git clone https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt

cd /home/zigbee/zigbee2mqtt

npm install

chown -R zigbee:zigbee /home/zigbee/zigbee2mqtt

22. Configure zigbee2mqtt

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

Reference config:  
  
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
  user: '*'
  password: '*'

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

zigbee2mqtt:info  2019-12-01T16:03:07: Logging to directory: '/home/zigbee/zigbee2mqtt/data/log/2019-12-01.18-03-05'
zigbee2mqtt:debug 2019-12-01T16:03:07: Loaded state from file /home/zigbee/zigbee2mqtt/data/state.json
zigbee2mqtt:info  2019-12-01T16:03:07: Starting zigbee2mqtt version 1.7.1 (commit #b459c35)
zigbee2mqtt:info  2019-12-01T16:03:07: Starting zigbee-herdsman...
zigbee2mqtt:debug 2019-12-01T16:03:07: Using zigbee-herdsman with settings: '{"network":{"panID":6754,"extendedPanID":[221,221,221,221,221,221,221,221],"channelList":[11],"networkKey":"HIDDEN"},"databasePath":"/home/zigbee/zigbee2mqtt/data/database.db","databaseBackupPath":"/home/zigbee/zigbee2mqtt/data/database.db.backup","backupPath":"/home/zigbee/zigbee2mqtt/data/coordinator_backup.json","serialPort":{"baudRate":115200,"rtscts":false,"path":"/dev/ttyS2"}}'
zigbee2mqtt:info  2019-12-01T16:03:10: zigbee-herdsman started
zigbee2mqtt:info  2019-12-01T16:03:10: Coordinator firmware version: '{"type":"zStack12","meta":{"transportrev":2,"product":0,"majorrel":2,"minorrel":6,"maintrel":3,"revision":20190109}}'
zigbee2mqtt:debug 2019-12-01T16:03:10: Zigbee network parameters: {"panID":6754,"extendedPanID":"0xdddddddddddddddd","channel":11}
zigbee2mqtt:info  2019-12-01T16:03:10: Currently 0 devices are joined:
zigbee2mqtt:warn  2019-12-01T16:03:10: `permit_join` set to  `true` in configuration.yaml.
zigbee2mqtt:warn  2019-12-01T16:03:10: Allowing new devices to join.
zigbee2mqtt:warn  2019-12-01T16:03:10: Set `permit_join` to `false` once you joined all devices.
zigbee2mqtt:info  2019-12-01T16:03:10: Zigbee: allowing new devices to join.
zigbee2mqtt:info  2019-12-01T16:03:10: Connecting to MQTT server at mqtt://localhost
zigbee2mqtt:info  2019-12-01T16:03:10: Connected to MQTT server
zigbee2mqtt:info  2019-12-01T16:03:10: MQTT publish: topic 'zigbee2mqtt/bridge/state', payload 'online'
zigbee2mqtt:info  2019-12-01T16:03:10: MQTT publish: topic 'zigbee2mqtt/bridge/config', payload '{"version":"1.7.1","commit":"b459c35","coordinator":{"type":"zStack12","meta":{"transportrev":2,"product":0,"majorrel":2,"minorrel":6,"maintrel":3,"revision":20190109}},"log_level":"debug","permit_join":true}'zigbee2mqtt:info  2019-12-01T16:01:50: MQTT publish: topic 'zigbee2mqtt/bridge/c                     onfig', payload '{"version":"1.7.1","commit":"b459c35","coordinator":{"type":"zS                     tack12","meta":{"transportrev":2,"product":0,"majorrel":2,"minorrel":6,"maintrel                     ":3,"revision":20190109}},"log_level":"debug","permit_join":true}'


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

