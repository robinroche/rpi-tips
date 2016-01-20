# rpi-tips
Tips for using a Raspberry Pi.

A lot of information on the Raspberry Pi (RPi) can be found at [eLinux.org](http://elinux.org/RaspberryPiBoard). Below are some basic information I found useful.

### Hardware

In order to get your RPi working, you need a few parts:
* A mini USB power supply,
* An HDMI cable, with a VGA/DVI adapter if you screen does not support HDMI,
* An SD card with a suitable Linux distribution on it, such as Raspbian,
* A keyboard and a mouse,
* A USB hub, if you need to connect more than 2 periphericals (e.g., when you need to connect a USB stick),
* An Ethernet cable, or a Wifi dongle, to enable web access and updates.


### Basic setup and commands

When you start your RPi for the first time, you should see the raspi-config tool. Use this tools to configure your time zone, keyboard (by default: qwerty) and locale settings. 

You can also activate remote ssh access. 

The default login and password are <code>pi</code> and <code>raspberry</code>, respectively.

Some basic commands:
* <code>startx</code>: launches the user interface
* <code>ifconfig</code>: displays network connectivity information
* <code>shutdown -r now</code>, or <code>reboot</code>: reboots
* <code>logout</code>: logout
* <code>date -s "Thu Aug  9 21:31:26 UTC 2012"</code>: manually sets the date to a specific value (input the UTC time)
* <code>sudo passwd root</code>: changes the root password
* <code>sudo -s</code>: access console as root
* <code>df -h</code>: get data on available space on disks

Learn more about Linux basics [here](http://raspberrywebserver.com/linux-basics/).


### Connecting to the web via a proxy

The following is valid for UTBM and with Raspbian, but a similar process should work in other cases. General procedures may also be found [here](http://hylix1.blogspot.com/2013/06/some-notes.html).

* Contact the CRI via  [GLPI](http://glpi.utbm.fr/), asking them for an IP for the RPi. You have to provide the MAC address of the RPi, using the <code>ifconfig</code> command in the terminal. 
* Once the IP is obtained, you can check that the RPi is correctly assigned to this address (using <code>ifconfig</code>) when it connects on the network. you can also ping other computers on the same network.
* You then need to configure the proxy, using the procedure described by VAXXi in [this post](http://www.raspberrypi.org/phpBB3/viewtopic.php?f=28&t=18634):
  * Go to folder <code>/etc/apt/apt.conf.d</code>,
  * Create a file named <code>10proxy</code>,
  * Add to this file the following command: <code>Acquire::http::Proxy "http://proxy.utbm.fr:3128";</code>
  * You may also check this address by downloading the [http://www.utbm.fr/proxy.pac proxy.pac] file. 
* Once this is done, you should be able to access the web. You can test this using an <code>apt-get update</code> command, or by pinging <code>www.google.com</code>, for example. If this doesn't work, try rebooting first. Although this should work for terminal commands, you may also need to configure some software manually with the correct proxy.

In some cases, e.g., for some <code>wget</code> commands, connecting might fail and give you a <code>Connection refused error</code>. Inputing the following command should solve the problem:

<code>export http_proxy='http://proxy.utbm.fr:3128'</code>

<code>export https_proxy='http://proxy.utbm.fr:3128'</code>

Similarly, for access to git:

<code>git config --global http.proxy http://proxy.utbm.fr:3128</code>

<code>git config --global https.proxy http://proxy.utbm.fr:3128</code>

ntp (automatic date update) does also not work behind the proxy. A workaround can be found [here](http://superuser.com/questions/307158/how-to-use-ntpdate-behind-a-proxy):

<code>sudo date -s "$(wget -S  "http://www.google.com/" 2>&1 | grep -E '^[[:space:]]*[dD]ate:' | sed 's/^[[:space:]]*[dD]ate:[[:space:]]*//' | head -1l | awk '{print $1, $3, $2,  $5 ,"GMT", $4 }' | sed 's/,//')"</code>


### Connecting to eduroam

At UTBM, the following parameters need to be used to connect to the eduroam wifi network [[source](https://blog.soton.ac.uk/pi/2013/11/19/eduroam-wifi-on-the-rpi/)]:
* WPA2-Enterprise (EAP)
* Encryption: CCMP
* EAP Method: TTLS
* Identity: <code>yourID@utbm.fr</code>
* Password: your password
* CA certificate: <code>/etc/ssl/certs/AddTrust_External_Root.pem</code>
* Inner auth: MSCHAPv2

This can be configured using the dedicated interface on the RPi's desktop. This has been tested with a Wi-Pi USB stick in building B. The package <code>wicd-curses</code> has been installed on the RPi.


### Preparing a SD card for Raspbian

Most 4+ GB SD cards can be used for running Raspbian, the default RPi distribution. This can be easily done using the NOOBS installer, and by following the instructions given in [this tutorial](http://elinux.org/RPi_Easy_SD_Card_Setup).

In some cases, e.g., when the SD card already has a distribution installed, regular formatting tools do not work and do not let you retrieve the entire space available on the card, but only a fraction of it. The solution here is to use Windows' <code>diskpart</code> tool:
* Launch a command window using Run > <code>cmd</code>,
* Type <code>diskpart</code>,
* List your disks using <code>list disk</code>,
* Select the proper disk using <code>select disk 1</code> (or the number corresponding to the disk you want to format; make sure you have the right one!),
* Type <code>clean</code>, to remove the existing partitions,
* Create a new primary partition using <code>create partition primary</code>,
* Format the card using <code>format fs=fat32 quick</code>,
* Assign a drive letter using <code>assign</code>.

If SDFormatter gives you an error indicating the drive is not supported, use another SD card reader (preferably an external, USB, one).


### Remotely connecting to the RPi

A first solution is to use SSH:
* Set a root password,
* Activate ssh using the raspi-config tool,
* Download [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/), run it and input the IP address of the RPi, with port 22 (default),
* Login,
* You should then be able to run commands in the terminal,
* When you are done, type <code>exit</code> or <code>logout</code>.


A second solution, that actually gives you an interface like [TeamViewer](http://www.teamviewer.com/), is to use a VNC:
* Install TighVNC, using <code>sudo apt-get install tightvncserver</code>,
* Run TightVNC, using <code>tightvncserver</code>, and set a password for remote access,
* Create a new file using <code>nano svnc.sh</code>, and add the following lines:
<code> #!/bin/sh
vncserver :0 -geometry 1920x1080 -depth 24 -dpi 96
</code>
* Set the file to execute, using <code>chmod +x svnc.sh</code>,
* Run this file, using <code>./svnc.sh</code>,
* From your computer, install and run TighVNCViewer, and use you RPi's IP address with port 5900 or 5901 (e.g., <code>192.168.0.1::5900</code>). You should be able to see the RPi's desktop.
* Note that this does not auto-start VNC, so you might have to start it via SSH first.
* Check out [this page](http://www.framboise314.fr/prenez-la-main-a-distance-sur-votre-raspberry-pi-avec-vnc/) (in French) for more details.


### Setting a hostname

Changing the default hostname (raspberrypi) of the RPi can be done in 4 steps:
* Run <code>sudo nano /etc/hosts</code>, and replace the current name at the line starting with <code>127.0.1.1</code> (and only this one) with the name you want to use,
* Run <code>sudo nano /etc/hostname</code>, and replace the current name with the same name you used in the previous step,
* Commit the changes using <code>sudo /etc/init.d/hostname.sh</code>,
* Reboot and check the hostname using the <code>hostname command</code>.


### Scheduling script runs with cron

The <code>cron</code> utility enables running instructions or scripts at a given frequency or time, e.g., every miunute or each monday at 7:30am. The basic way to change that is supposed to be through the <code>crontab -e</code>, but this does not seem to work properly.

I found the following procedure to work better:
* Edit the crontab file with <code>sudo nano /etc/crontab</code>,
* Add the script or instruction to run, such as, for a Python script run every 10 minutes as root: <code>*/10 * * * * root python /home/pi/script.py</code>,
* Save and exit.

To make sure cron is running at startup, use: <code>sudo update-rc.d cron enable 2 3 4 5</code>.


### Using GPIO pins

The following describes how to use the GPIO pins with a one-wire temperature sensor, the Dallas DS18B20:
* turn on the RPi and login,
* Load the drivers with the following commands: <code>sudo modprobe w1-gpio</code>, and <code>sudo modprobe w1-therm</code>,
* Move to the following folder: <code>cd /sys/bus/w1/devices/</code>, and list directories using <code>ls</code>,
* You should see a folder with a name containing mostly numbers, such as <code>28-000005209981</code>, move to this folder,
* Run this command: <code>cat w1_slave</code>,
* Two lines should appear; the last part of the second line contains the measured temperature (divide by 1000 to get the actual temperature).


You can also check Adafruit's [tutorial](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-11-ds18b20-temperature-sensing/overview) on the same topic. As explained on [this webpage](http://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/temperature/), the temperature can then be read using Python scripts.


The GPIO drivers are not loaded at boot, by default. This means that you will need to run the <code>modprobe</code> commands each time. To avoid that, use the following procedure [[source](http://raspberrypi.znix.com/hipidocs/topic_load_drivers_startup.htm)]:
* Edit the blacklist file via <code>sudo nano /etc/modprobe.d/raspi-blacklist.conf</code>,
* Comment the <code>i2c</code> and <code>spi</code> lines,
* Save and exit,
* Edit the modules file via <code>sudo nano /etc/modules</code>,
* Add the following lines: <code>i2c-dev</code>, <code>w1-gpio</code>, and <code>w1-therm</code>,
* Save and exit.


### Using an Adafruit 16x2 LCD screen

The following instructions are based on this [link](https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/usage).

The first step is to install I2C:
* Edit the following file <code>sudo nano /etc/modules</code> and add <code>i2c-bcm2708</code> and <code>i2c-dev</code> at the end
* Reboot
* Install the following packages: <code>sudo apt-get install python-smbus</code>, <code>sudo apt-get install i2c-tools</code>
* Edit this file: <code>sudo nano /etc/modprobe.d/raspi-blacklist.conf</code> and comment both lines ending with <code>bcm2708</code> with a <code>#</code>
* Reboot
* Check what addresses are being used: <code>sudo i2cdetect -y 1</code>
* If an Adafruit LCD Plate is connected, it should show up at <code>0x20</code>


The second step is to install the screen:
* Install the packages: <code>sudo apt-get install build-essential python-dev python-smbus python-pip git</code>
* Install the GPIO: <code>sudo pip install RPi.GPIO</code>
* Clone the git repository: <code>git clone https://github.com/adafruit/Adafruit_Python_CharLCD.git</code>
* Move to the folder: <code>cd Adafruit_Python_CharLCD</code>
* Install the LCD screen package: <code>sudo python setup.py install</code>



Finally, test the library:
* Move to the right folder: <code>cd examples</code>
* Run the example: <code>sudo python char_lcd_plate.py</code>


The screen should light up, and pushing the buttons should change the color.


### Using the camera module

The [Raspicam](http://www.raspberrypi.org/documentation/usage/camera/raspicam/README.md) package can be used with the camera module.
* Plug-in the camera as instructed [here](http://www.raspberrypi.org/documentation/usage/camera/),
* To capture stills with [raspistill](http://www.raspberrypi.org/documentation/usage/camera/raspicam/raspistill.md): <code>raspistill -o cam.jpg</code>
* To capture videos with [raspivid](http://www.raspberrypi.org/documentation/usage/camera/raspicam/raspivid.md): <code>raspivid -o video.h264 -t 10000</code> (10 seconds video)
* Create a [time-lapse](http://www.raspberrypi.org/documentation/usage/camera/raspicam/timelapse.md) using <code>cron</code>



### Interfacing the RPi

Interface with LabView: [blog post](http://etchingpathways.blogspot.fr/2013/09/labview-and-raspberry-pi-tcpip.html?sf22430439=1) and [code on GitHub](https://gist.github.com/gregpayne/6409958#file-tcpcommunicatorclient-py).



### RPi Programming

* Using Java on the RPi: [instructions from Oracle](http://www.oracle.com/technetwork/articles/java/raspberrypi-1704896.html) and [forum](http://www.raspberrypi.org/forums/viewforum.php?f=81).
* Installing Oracle's JAVA 7 JDK: <code>sudo apt-get install oracle-java7-jdk</code>
* To run JADE: Download and unzip [http://jade.tilab.com/ JADE], check whether it works using: <code>java -cp lib\jade.jar jade.Boot -gui</code>
