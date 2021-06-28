# Smarthome controller

Smarthome project based on Armbian 20.04 and BananaPi M2 Zero SBC

Setup

1. Download Armbian_21.05.1_Bananapim2zero_focal_current_5.10.34.img.xz from

https://redirect.armbian.com/bananapim2zero/Focal_current

2. Flash using Etcher https://etcher.io or

Rufus http://rufus.akeo.ie/downloads/rufus-2.11p.exe

3. Connect HDMI, keyboard, boot and login to console 

armbian / 1234

4. Set new root password

5. Choose default system command shell: choose bash

6. Press Ctrl-c to cancel user creation

7. Setup Wi-Fi network

armbian-config

Network->Wifi->select network->activate

8. Update system

apt update
apt upgrade

9. Set timezone

timedatectl set-timezone Europe/Kiev

10. Install NodeJS

curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_setup.sh

chmod +x nodesource_setup.sh

./nodesource_setup.sh

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

npm install -g --unsafe-perm node-red node-red-admin 

npm install -g pm2
pm2 start /usr/bin/node-red
pm2 save
pm2 startup

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

Use password hash from node-red-admin instead of "*"

apt install ufw -y
ufw allow 1880
reboot

13. Open Node-Red dashboard
http://192.168.43.253:1880

and enter login / password

14. Connect OLED to SBC

Pin		OLED	SBC

+3.3V	VCC		+3.3V
GND  	GND		GND
SCL		SCL		5
SDA		SDA		3

15. Enable I2C

armbian-config

System->Hardware->Select i2c0->Save->Back->Reboot

16. Install OLED status monitor

git clone https://github.com/makserge/WiringOP.git -b h3 
cd WiringOP
chmod +x ./build
sudo ./build
cd ..
ldconfig

git clone https://github.com/makserge/Raspberry-ssd1306.git
cd Raspberry-ssd1306

cc -o oled oled.c fontx.c -lwiringPi -lcrypt -lm -lrt -lpthread -DI2C  
bash ./test.sh 
cd ..

wget https://raw.githubusercontent.com/makserge/smarthome_controller/master/pushbuttons

chmod +x pushbuttons

./pushbuttons


nano /usr/local/bin/run23release.sh

#!/bin/bash

cpuTemp=`cat /sys/devices/virtual/thermal/thermal_zone0/temp`
cpuTempDegree=$((cpuTemp/1000))

freeRAM=`free | grep Mem | awk '{print (($4+$7)/$2 * 100)}'`
freeRAMRounded=`printf "%.1f" $freeRAM`

usedSpace=`df --output=pcent / | awk -F'%' 'NR==2{print $1}'`
freeSpace=$((100-$usedSpace))

cd /root/Raspberry-ssd1306

./oled r
./oled +1 `hostname -I | cut -d' ' -f1` 
./oled +2 "CPU temp: $cpuTempDegree C"
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
ExecStart=/root/Raspberry-ssd1306/pushbuttons
WorkingDirectory=/root
StandardOutput=inherit
StandardError=inherit
Restart=always
User=root

[Install]
WantedBy=multi-user.target


systemctl start pushbuttons
systemctl enable pushbuttons.service

17. Flash E18-MS1PA1-IPX Zigbee CC2530 2.4Ghz 100mW IPX Antenna IoT uhf Wireless Transceiver 2.4g Transmitter Receiver Module CC2530 PA
https://www.aliexpress.com/item/32640770110.html


Flashing proccess:

17.1 Download and unpack VLK DIY Multi Flasher (VLK_DIY_Multi_Flasher.rar) from

https://github.com/makserge/smarthome_controller/blob/master/E18-MS1PA1-IPX/VLK_DIY_Multi_Flasher.rar

17.2 Connect ESP32 board to Windows PC and start VLK DIY Multi_Flasher.exe
17.3 Switch to ZESP tab
17.4 Select ESP's COM port
17.5 Press "Write full flash" and wait for 3 minutes for finish
17.6 Connect IO21 and GND on ESP32
17.7 Connect Module to ESP32

ESP32	CC2530 module

5		RESET 	24
18		DC 		3
19		DD		4
GND		GND		1
3.3V	3.3V	2

17.8 Press Connect and check if module returned:

ChipID: A5
Chip: CC2530
Rev: 24
Size: 256k

17.9 Download firmware from goofyk(https://github.com/kirovilya)

https://github.com/makserge/smarthome_controller/blob/master/E18-MS1PA1-IPX/CC2530ZNP.hex

17.10 Flash firmware:

Choose CC2530ZNP.hex and press "Write"
Wait for Wrote Flash message (about 70s)

18. Connect CC2530 board to SBC

Pin  CC2530  SBC

+5V  +5V       +5V
GND  GND       GND
TX   P02       13
RX   P03       11


19. Enable UART2, UART3

armbian-config

System->Hardware->Select uart2, uart3->Save->Back->Reboot

nano /etc/rc.local

add 

stty -F /dev/ttyS2 115200
chmod 666 /dev/ttyS2
stty -F /dev/ttyS3 9600
chmod 666 /dev/ttyS3

before

exit 0

 
reboot

20. Check UART2, UART3

stty -F /dev/ttyS2

speed 115200 baud; line = 0;
-brkint -imaxbel

stty -F /dev/ttyS3

speed 9600 baud; line = 0;
-brkint -imaxbel

21. Add user zigbee

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

22. Install zigbee2mqtt 1.7.1
git clone --depth 1 --branch 1.7.1 https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt

git clone https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt

cd /home/zigbee/zigbee2mqtt

npm install

23. Configure zigbee2mqtt

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
homeassistant: true

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


24. Start zigbee2mqtt

chown -R zigbee:zigbee /home/zigbee/zigbee2mqtt

cd /home/zigbee/zigbee2mqtt
npm start

When started successfully, you will see:


                Zigbee2mqtt requires node version 10, you are running v14.17.1!

zigbee2mqtt:info  2021-06-28T10:28:01: Logging to directory: '/home/zigbee/zigbee2mqtt/data/log/2021-06-28.13-27-59'
zigbee2mqtt:debug 2021-06-28T10:28:01: Loaded state from file /home/zigbee/zigbee2mqtt/data/state.json
zigbee2mqtt:info  2021-06-28T10:28:01: Starting zigbee2mqtt version 1.7.1 (commit #b459c35)
zigbee2mqtt:info  2021-06-28T10:28:01: Starting zigbee-herdsman...
zigbee2mqtt:debug 2021-06-28T10:28:01: Using zigbee-herdsman with settings: '{"network":{"panID":6754,"extendedPanID":[221,221,221,221,221,221,221,221],"channelList":[11],"networkKey":"HIDDEN"},"databasePath":"/home/zigbee/zigbee2mqtt/data/database.db","databaseBackupPath":"/home/zigbee/zigbee2mqtt/data/database.db.backup","backupPath":"/home/zigbee/zigbee2mqtt/data/coordinator_backup.json","serialPort":{"baudRate":115200,"rtscts":false,"path":"/dev/ttyS2"}}'
zigbee2mqtt:info  2021-06-28T10:28:03: zigbee-herdsman started
zigbee2mqtt:info  2021-06-28T10:28:03: Coordinator firmware version: '{"type":"zStack12","meta":{"transportrev":2,"product":0,"majorrel":2,"minorrel":6,"maintrel":3,"revision":20190109}}'
zigbee2mqtt:debug 2021-06-28T10:28:03: Zigbee network parameters: {"panID":6754,"extendedPanID":"0xdddddddddddddddd","channel":11}
zigbee2mqtt:info  2021-06-28T10:28:03: Currently 1 devices are joined:
zigbee2mqtt:info  2021-06-28T10:28:03: kitchen_temp_humidity (0x00158d00020b824a): WSDCGQ01LM - Xiaomi MiJia temperature & humidity sensor (EndDevice)
zigbee2mqtt:warn  2021-06-28T10:28:03: `permit_join` set to  `true` in configuration.yaml.
zigbee2mqtt:warn  2021-06-28T10:28:03: Allowing new devices to join.
zigbee2mqtt:warn  2021-06-28T10:28:03: Set `permit_join` to `false` once you joined all devices.
zigbee2mqtt:info  2021-06-28T10:28:03: Zigbee: allowing new devices to join.
zigbee2mqtt:info  2021-06-28T10:28:03: Connecting to MQTT server at mqtt://localhost
zigbee2mqtt:info  2021-06-28T10:28:04: Connected to MQTT server
zigbee2mqtt:info  2021-06-28T10:28:04: MQTT publish: topic 'zigbee2mqtt/bridge/state', payload 'online'
zigbee2mqtt:info  2021-06-28T10:28:04: MQTT publish: topic 'zigbee2mqtt/bridge/config', payload '{"version":"1.7.1","commit":"b459c35","coordinator":{"type":"zStack12","meta":{"transportrev":2,"product":0,"majorrel":2,"minorrel":6,"maintrel":3,"revision":20190109}},"log_level":"debug","permit_join":true}'


25. Setup zigbee2mqtt as a daemon as root

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

● zigbee2mqtt.service - zigbee2mqtt
     Loaded: loaded (/etc/systemd/system/zigbee2mqtt.service; disabled; vendor >
     Active: active (running) since Sun 2021-06-27 22:02:31 EEST; 751ms ago
   Main PID: 2740 (npm)
      Tasks: 7 (limit: 901)
     Memory: 4.2M
     CGroup: /system.slice/zigbee2mqtt.service
             └─2740 npm

Jun 27 22:02:31 bananapim2zero systemd[1]: Started zigbee2mqtt.


Enable

sudo systemctl enable zigbee2mqtt.service


Reference

# Stopping zigbee2mqtt
sudo systemctl stop zigbee2mqtt

# Starting zigbee2mqtt
sudo systemctl start zigbee2mqtt

# View the log of zigbee2mqtt
sudo journalctl -u zigbee2mqtt.service -f

26. Setup watchdog

nano /root/blink.sh


#!/bin/sh -e

PIN=24

gpio mode $PIN out

while true; do
  gpio write $PIN 1
  sleep 0.5
  gpio write $PIN 0
  sleep 0.5
done

chmod +x /root/blink.sh

nano /etc/systemd/system/watchdogblink.service

[Unit]
Description=watchdogblink
After=network.target

[Service]
ExecStart=/root/blink.sh
WorkingDirectory=/root
StandardOutput=inherit
StandardError=inherit
Restart=always
User=root

[Install]
WantedBy=multi-user.target



Start daemon

systemctl start watchdogblink

Verify

systemctl status watchdogblink.service

● watchdogblink.service - watchdogblink
     Loaded: loaded (/etc/systemd/system/watchdogblink.service; disabled; vendo>
     Active: active (running) since Sun 2021-06-27 22:06:35 EEST; 12s ago
   Main PID: 3122 (blink.sh)
      Tasks: 2 (limit: 901)
     Memory: 348.0K
     CGroup: /system.slice/watchdogblink.service
             ├─3122 /bin/sh -e /root/blink.sh
             └─3177 sleep 0.5

Jun 27 22:06:35 bananapim2zero systemd[1]: Started watchdogblink.


Enable

sudo systemctl enable watchdogblink.service

27. Setup RTC

hwclock -f /dev/rtc0 -w

nano /etc/rc.local

add before 
exit 0

echo ds3231 0x68 > /sys/class/i2c-adapter/i2c-0/new_device
hwclock -f /dev/rtc0 -s


crontab -e

0 0 * * * timedatectl --adjust-system-clock; sudo hwclock -w

28. Install Home Assistant dependencies

sudo apt install python3-dev python3-pip python3-venv libffi-dev libssl-dev libjpeg-dev

29. Create Home Assistant user and group

sudo adduser --system homeassistant && addgroup homeassistant
sudo adduser homeassistant dialout

30. Create homeassistant folder and change permissions

sudo mkdir /opt/homeassistant
sudo chown homeassistant:homeassistant /opt/homeassistant

31. Change user

sudo su -s /bin/bash homeassistant

32. Create Python venv

cd /opt/homeassistant
python3 -m venv /opt/homeassistant
source bin/activate

33. Install Home Assistant

python3 -m pip install wheel
pip3 install homeassistant

34. Create Home Assistant config folder

mkdir /opt/homeassistant/config

35. Add Home Assistant to autostart

exit

nano /etc/systemd/system/hass.service

[Unit]
Description=Home Assistant
After=network.target

[Service]
Type=simple
User=homeassistant
ExecStart=/opt/homeassistant/bin/hass -c /opt/homeassistant/config --log-file /opt/homeassistant/hass.log

[Install]
WantedBy=multi-user.target

36. Enable autostart for Home Assistant

sudo systemctl --system daemon-reload
sudo systemctl enable hass && systemctl start hass

37. Check status

systemctl status hass


● hass.service - Home Assistant
     Loaded: loaded (/etc/systemd/system/hass.service; enabled; vendor preset: >
     Active: active (running) since Sun 2021-06-27 11:53:41 UTC; 1h 47min ago
   Main PID: 1222 (hass)
      Tasks: 8 (limit: 901)
     Memory: 100.3M
     CGroup: /system.slice/hass.service
             └─1222 /opt/homeassistant/bin/python3 /opt/homeassistant/bin/hass >

38. Open Browser on http://myipaddress:8123 and finish onboarding

https://www.home-assistant.io/getting-started/onboarding/

39. Setup zigbee2mqtt integration

nano /opt/homeassistant/config/configuration.yaml
 
add 
 
mqtt:
  discovery: true
  broker: 127.0.0.1
  will_message:
    topic: 'homeassistant/status'
    payload: 'offline'

systemctl stop hass
systemctl start hass

Open
http://xxx.xxx.xxx.xxx:8123/config/integrations

Configure->Re-configure MQTT->Set username and password->Next->Submit->Finish

40. Configure Zigbee temperature and humidity sensor

http://xxx.xxx.xxx.xxx:8123/config/mqtt

Listen to a topic zigbee2mqtt/kitchen_temp_humidity

Check if something like this was received

Message 0 received on zigbee2mqtt/kitchen_temp_humidity at 1:44 PM:
{
    "battery": 100,
    "voltage": 3025,
    "temperature": 29.33,
    "humidity": 46.63,
    "linkquality": 148
}
Edit Dashboard->Unused entries->Check "kitchen_temp_humidity_temperature" and "kitchen_temp_humidity_humidity"->Add card
Developer->click "kitchen_temp_humidity_temperature" and set

friendly_name: Temperature

click sensor.kitchen_temp_humidity_humidity and set

friendly_name: Humidity
/*

nano /opt/homeassistant/config/sensors.yaml


sensor:
    - platform: mqtt
      name: 'Kitchen temperature'
      state_topic: 'zigbee2mqtt/kitchen_temp_humidity'
      unit_of_measurement: '°C'
      value_template: "{{ value_json.temperature | round(1) }}"
      availability_topic: "zigbee2mqtt/bridge/state"
      device_class: "temperature"
    - platform: mqtt
      name: 'Kitchen humidity'
      state_topic: 'zigbee2mqtt/kitchen_temp_humidity'
      unit_of_measurement: '%'
      value_template: "{{ value_json.humidity | round(0) }}"
      availability_topic: "zigbee2mqtt/bridge/state"
      device_class: "humidity"

systemctl restart hass 
*/

40. Configure Modbus RTU

nano /opt/homeassistant/config/configuration.yaml
 
add 

sensor: !include sensors.yaml

after 
scene: !include scenes.yaml

add

modbus:
  name: modbus
  type: serial
  baudrate: 9600
  bytesize: 8
  method: rtu
  parity: N
  port: /dev/ttyS3
  stopbits: 1

nano /opt/homeassistant/config/sensors.yaml  

chown homeassistant.nogroup sensors.yaml

41. Add light switch


modbus:
  name: modbus
  type: serial
  baudrate: 9600
  bytesize: 8
  method: rtu
  parity: N
  port: /dev/ttyS3
  stopbits: 1
  lights:
      - name: Light 4
        slave: 21
        address: 3
        write_type: coil




Edit dashboard->Add card->By Entity->Select "Light 4"->Continue->Add to lovelace UI	

41. Add switch

modbus:
  name: modbus
  type: serial
  baudrate: 9600
  bytesize: 8
  method: rtu
  parity: N
  port: /dev/ttyS3
  stopbits: 1
  switches:
      - name: Switch 4
        slave: 21
        address: 3
        write_type: coil
		
Edit dashboard->Add card->By Entity->Select "Switch 4"->Continue->Add to lovelace UI

42. Add binary sensor

modbus:
  name: modbus
  type: serial
  baudrate: 9600
  bytesize: 8
  method: rtu
  parity: N
  port: /dev/ttyS3
  stopbits: 1
  binary_sensors:
    - name: "binary_sensor1"
      address: 3
      scan_interval: 5
      slave: 21
	  
	  
43. Setup Modbus CLI
 
pip install modbus_cli	

modbus -s 21 -b 9600 -p 1 -P n -v /dev/ttyS3 1 c@3 = read coil at address 3

