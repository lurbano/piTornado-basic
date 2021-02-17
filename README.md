# piTornado-basic
Template for using tornado server on raspberry pi
* Lensyl Urbano
* https://montessorimuddle.org


# Set up Raspberry Pi

## Install OS
### Create image on the SD card:
 (make image using Raspberry Pi Imager: https://www.raspberrypi.org/software/)

### Setup ssh, wifi, and usb connection
 Working on the boot directory of the SD Card

#### 1) ssh
[copy] or create empty file ***ssh*** on SD Card's boot directory
* Filename: *ssh*

#### 2) wpa_supplicant.conf
[edit] or create file for wifi connection and copy to boot directory of Pi:
* File name: *wpa_supplicant.conf*
* Change: `networkName` and `yourPassword`

The file should look like:

*wpa_supplicant.conf*
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
 ssid="networkName"
 psk="yourPassword"
}
```

#### 3) USB connection
[copy] over or update files on the SD Card for usb connection (https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget)

***config.txt***: Add `dtoverlay=dwc2` as the last line.
 ```
dtoverlay=dwc2
```

***cmdline.txt***: Insert:
``` modules-load=dwc2,g_ether```
after `rootwait` (e.g. `rootwait modules-load=dwc2,g_ether`).


## Set up Pi

### Connect to Pi

Plug Pi into Laptop USB then once pi has booted up:

*Option 1: Windows*: Login with (putty: https://www.putty.org/):
* PuTTY Host/IP: raspberrypi.local
* Port: 22
* Username: pi
* Password: raspberry

*Option 2: Mac or Linux*: use Terminal app, which is built in, for the command line:
```console
ssh pi@raspberrypi.local
```

*NOTE: Troubleshooting*: you may have to remove the **~/.ssh/known_hosts** file if you find yourself logging in to the wrong pi or unable to connect.
```console
rm .ssh/known_hosts
```


### update Pi
Once you're logged into the Pi
 ```console
sudo apt-get update
sudo apt-get upgrade
```

### REBOOT pi
 ```console
sudo reboot
```
```

# Installing this software: r
From your home directory clone the github repository.
```console
git clone https://github.com/lurbano/piTornado-basic.git
```

# Setting up Server
## Install Tornado Webserver

Setting up the tornado server used for Websockets
```console
sudo pip3 install tornado
```

### Starting server
Go to the folder *~/piTornado/webServer/* and run the command
```console
sudo python3 server.py
```

### The webpage
The webpage will be at the pi's ip address (which should be printed to the screen when you start the server) and on port 8060 so if your ip address is 192.168.1.234, open up your browser and go to:
> http://192.168.1.234:8060

### Starting up on boot
** IMPORTANT **: the directory with the files needs to be in the pi home directory (e.g. */home/pi/rpi-led-strip*) with this setup. You can change this but be sure to put the full path to the commands. (From: https://learn.sparkfun.com/tutorials/how-to-run-a-raspberry-pi-program-on-startup)

EDIT */etc/rc.local* (the easy way)
```console
sudo nano /etc/rc.local
```

ADD THE LINE (before `exit 0` ).
```
sudo /usr/bin/python3 /home/pi/piTornado-basic/webServer/server.py  2> /home/pi/rpi-led-strip/error.log &
```

Save and exit (Ctrl-S and Ctrl-X) and then restart the Pi from the command line:
```console
sudo reboot
```


### If you need to kill the server
* https://unix.stackexchange.com/questions/104821/how-to-terminate-a-background-process
```console
pgrep -a python3
```
* this will give you the process id, the name line of the command, and a number 'nnn'. Find the one that has 'python3 server.py'. To kill use:
```console
sudo kill nnn
```


# [EXAMPLE] Adding things to be controlled by the webpage
Say you want to add a button that makes the LED's blue

## Add hello world button
*webServer/templates/index.html*: Add HTML for button and a span where we will put the response from the server.
```HTML
<input type="button" id="hello" value="Hello World">
<span id="HelloResponse"></span>
```

## Add javascript
To listen for when someone clicks the blueButton:
*webserver/static/ws-client.js* near bottom of file
```js
$("#hello").click(function(){
    let msg = '{"what": "hello"}';
    ws.send(msg);
});
```

Here we're sending the dict {"what": "hello"} to the server.

We also need a method to deal with the response from the server. IN the ws.onmessage function add:
```js
if (sData.info == 'hello'){
  r = sData.reply.toString();
  $("#timeLeft").html(r);
}
```


## Have the server act
It has to figure out what to do when it gets the message: msg = {"what": "blueButton"} in *webserver/server.py* around line 76. Here it cancels anything already going on (ledPix.cancelTask) and calls the method .blue() from the ledPix instance of the ledPixels class (you'll most often need to add your own method (see next step)).
```.py
			if msg["what"] == "hello":
				r = 'Say what?'
				self.write_message({"info": "hello", "reply":r})
```


# Refs:
OLED:
* http://codelectron.com/setup-oled-display-raspberry-pi-python/
* https://learn.adafruit.com/adafruit-pioled-128x32-mini-oled-for-raspberry-pi/usage
