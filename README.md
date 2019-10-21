# Smarthome controller

Smarthome project based on Armbian and Nano PI Neo SBC

Setup

1. Download Armbian_5.98.191021_Nanopineo_Debian_buster_dev_5.3.7_minimal.img from

https://dl.armbian.com/nanopineo/Debian_buster_dev_minimal_nightly.7z

2. Flash using Etcher https://etcher.io or

Rufus http://rufus.akeo.ie/downloads/rufus-2.11p.exe

3. Boot and login to console using SSH

root / 1234

4. Set new root password

Changing password for root.
Current password:
Retype new password:

5. Create new user

You are using an Armbian nightly build meant only for developers to provide
constructive feedback to improve build system, OS settings or user experience.
If this does not apply to you, STOP NOW!. Especially don't use this image for
daily work since things might not work as expected or at all and may break
anytime with next update. YOU HAVE BEEN WARNED!

This image is provided AS IS with NO WARRANTY and NO END USER SUPPORT!.

Creating a new user account. Press <Ctrl-C> to abort

Please provide a username (eg. your forename): smarthome
Trying to add user smarthome
Adding user `smarthome' ...
Adding new group `smarthome' (1000) ...
Adding new user `smarthome' (1000) with group `smarthome' ...
Creating home directory `/home/smarthome' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for smarthome
Enter the new value, or press ENTER for the default
        Full Name []: smarthome
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y

Dear smarthome, your account smarthome has been created and is sudo enabled.
Please use this account for your daily work from now on.

6. Update system

apt update
apt upgrade

7. Setup Wi-Fi network (RTL8188EUS)

nmtui

Activate a connection
select network from list and choose activate
enter password and press ok

8. Install armbian-config

apt install armbian-config

9. Set timezone

armbian-config

Personal->Timezone->Europe->Kiev->Ok

10. Enable UART1-UART2

armbian-config

System->Hardware->uart1,uart2->Save->Back->Reboot

11. Install NodeJS

curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -

apt install -y nodejs

12. Install Mosquitto

apt install mosquitto

mosquitto_passwd -c /etc/mosquitto/passwd smarthome
Password:

nano /etc/mosquitto/conf.d/default.conf
add

allow_anonymous false
password_file /etc/mosquitto/passwd


systemctl restart mosquitto

13. Install Node-red

npm install -g --unsafe-perm node-red

npm install -g pm2
pm2 start /usr/bin/node-red
pm2 save
pm2 startup

npm install -g --unsafe-perm node-red-admin

node-red-admin hash-pw


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
} 

14. Open Node-Red dashboard
http://192.168.43.253:1880

and enter login / password

15. Connect CC2530 board to SBC

Pin  CC2530  SBC

+5V  +5V       +5V
GND  GND       GND
TX   P02       8
RX   P03       10


16. Setup UART2 

nano /etc/rc.local

add
stty -F /dev/ttyS2 115200
chmod 666 /dev/ttyS2

before
exit 0

 
reboot

17. Check UART2

stty -F /dev/ttyS2

speed 115200 baud; line = 0;
-brkint -imaxbel

18. Add user zigbee

adduser zigbee
 
19. Install zigbee2mqtt

npm install -g node-gyp
apt install -y git make g++ gcc python-dev

git clone https://github.com/Koenkk/zigbee2mqtt.git /home/zigbee/zigbee2mqtt
chown -R zigbee:zigbee /home/zigbee/zigbee2mqtt

chown -R zigbee:zigbee /home/zigbee/zigbee2mqtt

20. Login as zigbee user

cd /home/zigbee/zigbee2mqtt

npm install

21. Configure zigbee2mqtt

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


22. Start zigbee2mqtt

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

23. Setup zigbee2mqtt as a daemon as root

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

