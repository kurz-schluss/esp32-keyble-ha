# esp32-keyble-ha
<h1>ESP32 port of the keyble library</h1>
<h2>What is this for?</h2>

The Eqiva eQ3 is a cheap but reliable smart lock that can be fitted to many doors. It has bluetooth connectivity and needs no external power supply. 
But there is no direct interface to integrate this lock in your home automation setup. This port of the keyble library adds a bluetooth to WIFI bridge
on a small ESP32 IOT device. Just place the ESP32 near the lock and add it to your local WLAN.

The lock can be controlled via MQTT and integrated into Home Assistant.

Thanks go to <a href="https://github.com/RoP09">RoP09</a>, <a href="https://github.com/tc-maxx">tc-maxx</a>, <a href="https://github.com/henfri">henfri</a>, <a href="https://github.com/MariusSchiffer">MariusSchiffer</a> and of course <a href="https://github.com/oyooyo">oyooyo</a> for their brillant work!

This is a fork from <a href"https://github.com/mccoy88f/esp32-keyble-ha">esp32-keyble-ha</a> from <a href="https://github.com/mccoy88f">mccoy88f</a> with some improvements.

Fiddled around a bit to get it to run more stable.
IF you have problem with wifi reconnect before flash in send this command in your terminal/shell changing port with yours:
esptool.py --chip esp32 --port /dev/cu.usbserial-0001 erase_flash
(search on web how to install esptool)

#UPDATE 26-08-2023
Add command in mqtt to reboot device (using payload 7)
You will also find in ha package a button for that

#UPDATE 08-08-2023
Fixed library for working with latest VScode and Platformio Version.
Compile and work:
A testing binary is attached in main folder (firmware.bin) for a 4MB ESP32 BOARD (i suggest to compile your own version).

Some changes/additions made:

- RSSI value for BLE connection
- battery state
- toggle function for lock
- AP-Mode and config Portal via AutoConnect
- MQTT endpoints for state, task and battery 
- OTA update to upload bin files
- removed hardcoded credentials
- serial outputs in english
- boot button on the ESP32 board toggles the lock
- changed partion table
- NTP time synchronization for the auto-lock function that can be configured via the app with NTP server and time zone

Step by step setup:

#DEVICE SIDE SETUP
- Erase the flash first because of SPIFFS usage and maybe stored old WiFi credentials.
- Upload the project (I use Platformio).
- Connect to the ESP32's WiFi network.
- By default the SSID is "KeyBLEBridge" with default password "eqivalock". For security reasons you have to connect within 5 minutes after startup.
- After you have connected to the network, go to 192.168.4.1 with you browser.
- At rootlevel you will just see a simple webpage.
- Click on the gear to connect to your WiFi Network.
- Now you will see the AutoConnect portal page.
- Go to "Configure new AP" to enter your credentials.
- AutoConnect scans for networks in reach, just choose the one you want to connect to.
- I recommend you to disable DHCP and give the bridge a static IP.
- The ESP32 reboots, so you have to access it with the new given IP.
- Login with default "admin"/"esp32"
- Enter MQTT, NTP server, time zone and KeyBLE credentials.
- IMPORTANT: for direct use in home assistant use "homeassitant/lock/door" as MQTT Topic
- IF YOU DO NOT HAVE KeyBLE credential you need the original QRCode in the eq3 box and a linux pc or raspberry where install via NPM command keyble package (info here https://github.com/oyooyo/keyble) and then use keyble-registeruser command.
- After saving the credentials click on "Reset" to reboot.
- You well be redirected to the main page of AutoConnect.
- Click on "Home" to see the entered credentials.
- Now everthing is set up and the ESP publishes its states to the given MQTT Broker.
- It publishes the state once on startup and has then to be triggert from outside.

The bridge publishes to the given topic you entered at setup.

#HOME ASSISTANT SIDE SETUP
- Put "eq3_bridge_package_de.yaml" in packages folder of Home Assistant
  (if you never use packages in Home Assistant follow also this step:=
  - create a folder named “packages” in the same location where your configuration.yaml resides.
  - then in configuration.yaml make it look like something similar to this (add the notated line under “homeassistant:”):
    homeassistant:
      packages: !include_dir_named packages
  - reboot home assistant
  - you will find lock, sensor, and also some notification (like low battery). Status of lock is readed on Home Assistant boot and also after 5 minutes of "unknown" battery status.
My package is german, if you want to change take care of names or something will broke. If someone wants to translate send a pull request.
Also the package is for 1 lock only, you need to modify it if you have more lock of equiva and more esp32 bridge.

MQTT Endpoints are:

- /battery true for good, false for low
- /command 1 for status, 2 to unlock, 3 to lock, 4 to open and 5 to toggle)
- /rssi signalstrength
- /state as strings; locked, unlocked, open, moving, timeout
- /task working; if the lock has already received a command to be executed, waiting; ready to receive a command

The /task endpoint is usefull, because the ESP toggles between WiFi and BLE connections. If the bridge is connected to WiFi and recieves a command via MQTT, it disables WiFi, connects to the lock via BLE, reconnects after the BLE task has finished and publishes the new state to the mqtt broker.

TODO
- register user feature
- error handling
- more endpoints like IP, uptime, etc.
- a command queue topic to get commands while the bridge was busy
- a nice 3D printed housing

Beside the ESP32 solutions, I have a modded verison of oyooyo's first implementaion running on a Pi Zero, with 5 locks permanently connected at the same time a (Batterys last around 2 month). The locks are managed with Node Red and integrated in a Loxone Smart Home with a Doorbird and an external RFID reader. If I have time I will upload this to github too.


Have fun!
