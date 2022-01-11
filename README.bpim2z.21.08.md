# Smarthome controller

Smarthome project based on Armbian 21.08 and BananaPi M2 Zero SBC

Setup

1. Download Bananapim2zero_bullseye_current_5.10.60.img.xz from

https://armbian.hosthatch.com/archive/bananapim2zero/archive/Armbian_21.08.1_Bananapim2zero_bullseye_current_5.10.60.img.xz

2. Flash using Etcher https://etcher.io or

Rufus http://rufus.akeo.ie/downloads/rufus-2.11p.exe

3. Connect HDMI, keyboard, boot and login to console 

root / 1234

4. Set new root password

5. Choose default system command shell: choose bash

6. Press Ctrl-c to cancel user creation

7. Setup Wi-Fi and wired network

armbian-config

Network->Wifi->select network->activate

Network->IP->eth0->Static

Set Address, Netmask, Gateway, DNS->Ok->Back->Exit


8. Update system

apt update
apt upgrade

9. Set timezone

timedatectl set-timezone Europe/Kiev

10. Disable automatic updates

sudo nano /etc/apt/apt.conf.d/20auto-upgrades

replace

APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";

to 

APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";

11. Install Mosquitto

apt install mosquitto -y

mosquitto_passwd -c /etc/mosquitto/passwd smarthome
Password:

nano /etc/mosquitto/conf.d/default.conf
add

allow_anonymous false
password_file /etc/mosquitto/passwd


systemctl restart mosquitto

12. Connect OLED to SBC

Pin		OLED	SBC

+3.3V	VCC		+3.3V
GND  	GND		GND
SCL		SCL		5
SDA		SDA		3

13. Enable I2C

armbian-config

System->Hardware->Select i2c0->Save->Back->Reboot

14. Install OLED status monitor

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

freeRAM=`free | grep Mem | awk '{print $7/$2 * 100.0}'`
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


nano  /usr/local/bin/run23long.sh

#!/bin/bash

cd /root/Raspberry-ssd1306
./oled r
./oled +1 "" 
./oled +2 "Shutting down" 
./oled s
sleep 5
./oled r
./oled s

shutdown -h now


chmod +x /usr/local/bin/run23long.sh

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


15. Enable UART2, UART3

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

16. Check UART2, UART3

stty -F /dev/ttyS2

speed 115200 baud; line = 0;
-brkint -imaxbel

stty -F /dev/ttyS3

speed 9600 baud; line = 0;
-brkint -imaxbel

17. Setup watchdog

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

18. Setup RTC

hwclock -f /dev/rtc0 -w

nano /etc/rc.local

add before 
exit 0

echo ds3231 0x68 > /sys/class/i2c-adapter/i2c-0/new_device
hwclock -f /dev/rtc0 -s


crontab -e

0 0 * * * timedatectl --adjust-system-clock; sudo hwclock -w

19. Temporary increase SWAP to 1GB

rm -rf /dev/zram2
zramctl --reset /dev/zram2
zramctl --find --size 256M
mkswap /dev/zram2
swapon /dev/zram2 --priority 2

rm -rf /dev/zram3
zramctl --reset /dev/zram3
zramctl --find --size 256M
mkswap /dev/zram3
swapon /dev/zram3 --priority 3

rm -rf /dev/zram4
zramctl --reset /dev/zram4
zramctl --find --size 256M
mkswap /dev/zram4
swapon /dev/zram4 --priority 4


20. Install Home Assistant dependencies

sudo apt install python3-dev python3-pip python3-venv libffi-dev libjpeg-dev

21. Create Home Assistant user and group

sudo adduser --system homeassistant && addgroup homeassistant
sudo adduser homeassistant dialout

22. Create homeassistant folder and change permissions

sudo mkdir /opt/homeassistant
sudo chown homeassistant:homeassistant /opt/homeassistant

23. Change user

sudo su -s /bin/bash homeassistant

24. Create Python venv

cd /opt/homeassistant
python3 -m venv /opt/homeassistant
source bin/activate

25. Add Rust

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

26. Install Home Assistant

python3 -m pip install wheel

pip3 install homeassistant

27. Create Home Assistant config folder

mkdir /opt/homeassistant/config

28. Add Home Assistant to autostart

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

29. Enable autostart for Home Assistant

sudo systemctl --system daemon-reload
sudo systemctl enable hass && systemctl start hass

30. Check status

systemctl status hass


● hass.service - Home Assistant
     Loaded: loaded (/etc/systemd/system/hass.service; enabled; vendor preset: >
     Active: active (running) since Sun 2021-06-27 11:53:41 UTC; 1h 47min ago
   Main PID: 1222 (hass)
      Tasks: 8 (limit: 901)
     Memory: 100.3M
     CGroup: /system.slice/hass.service
             └─1222 /opt/homeassistant/bin/python3 /opt/homeassistant/bin/hass >

31. Open Browser on http://myipaddress:8123 and finish onboarding

https://www.home-assistant.io/getting-started/onboarding/


32. Configure Modbus RTU

nano /opt/homeassistant/config/configuration.yaml
 
add 

modbus: !include modbus.yaml

after 
scene: !include scenes.yaml


nano /opt/homeassistant/config/modbus.yaml  

name: modbus
type: serial
baudrate: 9600
bytesize: 8
method: rtu
parity: N
port: /dev/ttyS3
stopbits: 1
timeout: 4
delay: 3


33. Add light switch

nano /opt/homeassistant/config/modbus.yaml  

name: modbus
type: serial
baudrate: 9600
bytesize: 8
method: rtu
parity: N
port: /dev/ttyS3
stopbits: 1
timeout: 4
delay: 3
lights:
    - name: Light 4
      slave: 21
      address: 3
      write_type: coil


Edit dashboard->Add card->By Entity->Select "Light 4"->Continue->Add to lovelace UI	

34. Add switch

nano /opt/homeassistant/config/modbus.yaml  

name: modbus
type: serial
baudrate: 9600
bytesize: 8
method: rtu
parity: N
port: /dev/ttyS3
stopbits: 1
timeout: 4
delay: 3
switches:
    - name: Switch 4
      slave: 21
      address: 3
      write_type: coil
      verify:
      scan_interval: 5

Edit dashboard->Add card->By Entity->Select "Switch 4"->Continue->Add to lovelace UI

35. Add binary sensor

nano /opt/homeassistant/config/modbus.yaml  

name: modbus
type: serial
baudrate: 9600
bytesize: 8
method: rtu
parity: N
port: /dev/ttyS3
stopbits: 1
timeout: 4
delay: 3
binary_sensors:
  - name: "binary_sensor1"
    address: 3
    scan_interval: 5
    slave: 21

