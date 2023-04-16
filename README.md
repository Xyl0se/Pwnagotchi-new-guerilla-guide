# Pwnagotchi-new-guerilla-guide
Updated version of panoptyks 2022 guerilla guide

## Preface

This work is based on the **guerrila guide to installing pwnagotchi [1.5.5/2022]**. For reference see [this reddit post](https://www.reddit.com/r/pwnagotchi/comments/sl2rv1/guerrilla_guide_to_pwnagotchi_v1552022/) and [pastebin](https://pastebin.com/8a3Rbyx6) of [panoptyk](https://www.reddit.com/user/panoptyk/).

Since panoptyks original writeup some things have changed and I ran into several problems while setting up my pwnagotchi for the first time. Some bugs are still unresolved, see Section *known bugs*.

However, the normal installation guide from [the official website](https://pwnagotchi.ai) is still valid by and large, so this document will mostly provide details regarding deviations/addons to the standard installation procedure.

The specific hardware I myself have been using for my build is this:

- Raspberry Pi Zero WH (RP0W)
- Waveshare 2.13 V3
- PiSugar 2 (2 LED-version)
- Samsung Evo 32GB micro SD

I will do my best to reference and credit every source, please add sources where missing and/or drop me a hint. Majority of this document has been created in 2023-04, things are moving fast and surely another year later there will be other stuff broken, and other fixes available.

Not every section from the original guide has been used in this document, the omitted text can be found in the file *original_notes*.

## 1. Flash the current image

### 1.1 Download

- [balena etcher](https://www.balena.io/etcher/)
- [pwnagotchi v1.5.5](https://github.com/evilsocket/pwnagotchi/releases/tag/v1.5.5)

### 1.2 write the image to a micro sd card with balena etcher

(for details see pwnagotchi.ai)

## 2. Basic Connectivity (SSH, FTP, Connection Sharing)

### 2.1 Connect to PC

Because I am running Windows 11 as a daily driver, this section will cover only Windows. For establishing a connection on Linux or MacOS refer to the [official guide](https://pwnagotchi.ai/configuration/).

1. connect RP0W data port to pc (Micro-USB to USB A)
2. wait for the device to boot up for the first time (20+ Minutes)
3. Check Device Manager for COM-Port - [as described here](https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget#if-you-are-using-windows-as-the-host-machine-2570572)
4. If Windows didn't install the RP0W as an "Ethernet Gadget", download the corresponding driver on [Windows Update](https://www.catalog.update.microsoft.com/Search.aspx?q=USB+RNDIS%20Gadget)
5. Unpack the CAB
6. In Device Manager select the COM-Device and update the driver with the one from the CAB
7. make sure to check in Network Devices for the Interface and configure TCP/IP v4 to use 10.0.0.01 as its IP-address, 255.255.255.0 as Subnet and 10.0.0.1 as Gateway.

### 2.1 SSH-connection

Open PowerShell as Administrator

```bash
ssh pi@10.0.0.2
```

default password is raspberry, it is generally recommended to immediately change it:

```bash
passwd
```

**if you get the WARNING:** `REMOTE HOST IDENTIFICATION HAS CHANGED!`, go to `C:\Users\{user}\.ssh`, open `known_hosts` and comment out (#) every line; save
connect again with pi (`ssh pi@10.0.0.2`); confirm/authorize  (yes in terminal)

### 2.2 establish ftp support

this is optional, but makes installation way easier (imho). On Windows, I use
[WinSCP](https://winscp.net/eng/download.php), ymmv.

First, enable password login for root (unsafe, should be disabled again after setup has finished).

```bash
passwd #change pi's password
sudo su
passwd root #change root's password
```

Then, enable SFTP support by executing

```bash
sudo nano /etc/ssh/sshd_config
```

uncomment and change the line `#PermitRootLogin prohibit-password` to `PermitRootLogin yes`

then restart ssh by executing:

```bash
service ssh restart
```

login via FTP using:

```bash
host: 10.0.0.2
username: root
password: *password*
port: 22
```

### 2.3 solve DNS issues

```bash
sudo nano /etc/resolv.conf
```

change the entry behind *nameserver* "127.0.0.1" to "8.8.8.8".

**Warning** this is only temporary and will be overwritten on each reboot. There are several different fixes for this, see [this thread](https://github.com/evilsocket/pwnagotchi/issues/859). Short summary:

- adding dns-nameservers 8.8.8.8 under the gateway line in /etc/network/interfaces.d/usb0-cfg
- add: "server=8.8.8.8@usb0" to /etc/dnsmasq.conf
- systemctl disable dnsmasq
- sudo chattr +i /etc/resolv.conf makes file immutable (make sure beforehand there is 8.8.8.8 or 1.1.1.1 insted of 127.0.0.0)

### 2.4 Internet connection sharing for win10/11

 The official script is named "win_connection_share.ps1" and can be copied via ftp from /usr/local/src/pwnagotchi/scripts once connected.

Execute this in Powershell:

```powershell
  1.) .\win_connection_share.ps1 -SetPwnagotchiSubnet
  2.) # Reboot Windows
  3.) .\win_connection_share.ps1 -EnableInternetConnectionSharing
```

*if applicable: enable mobilehotspot on pc and turn off power saving mode for your hotspot or your wireless interface.*

## 3 Upload initial config.toml

1. Prepare your config.toml according to the official guide.
2. connect through ftp and upload *your config.toml* to /etc/pwnagotchi/
3. make directory /etc/pwnagotchi/custom-plugins for custom plugins to add to that directory later.

## 4 Enable Bluetooth connection

### 4.1 change BT settings in stock/default config

This is going to be dependent on your Bluetooth device, but necessary to enable internet access when connected to your Bluetooth device (do it if you didn't already supply your premade/working config in the previous step)

change following options:

- [ ] change this to iPhone, because that's my config

```toml
main.plugins.bt-tether.enabled = true
main.plugins.bt-tether.devices.android-phone.enabled = true
main.plugins.bt-tether.devices.android-phone.search_order = 1
main.plugins.bt-tether.devices.android-phone.mac = "CH:AN:GE:ME:HE:RE" #phone: settings-> about device -> status "bluetooth address"  
main.plugins.bt-tether.devices.android-phone.ip = "192.168.44.44"
main.plugins.bt-tether.devices.android-phone.netmask = 24
main.plugins.bt-tether.devices.android-phone.interval = 1
main.plugins.bt-tether.devices.android-phone.scantime = 10
main.plugins.bt-tether.devices.android-phone.max_tries = 0
main.plugins.bt-tether.devices.android-phone.share_internet = true
main.plugins.bt-tether.devices.android-phone.priority = 1
```

then reboot pwnagotchi, either via ssh --> `sudo reboot now` or through the [web UI in your browser](http://10.0.0.2:8080).

### 4.2 pair pwnagotchi with phone (IMPORTANT!)

make sure BT and BT tethering are activated on your phone. keep phone unlocked, pair phone.

If for some reason pwnagotchi stops connecting to your phone after some time, or wont connect at all, try this:

```bash
# RP0W data port <--> pc
ssh pi@10.0.0.2
sudo su
bluetoothctl
scan on
discoverable on
paired-devices # copy device adress
untrust *device adress*  #run this command a few times
remove *device adress*   #run this command a few times
paired-devices #make sure list is empty, if not- run previous command until it is empty
pair *device adress* #*In short time (maybe not immediately) you will be prompted on the phone to allow connection from your pwnagotchi hostname- pair*
trust *device adress*
exit
```

after that, open cmd window with ssh session(!), and

```bash
ping google.com
```

if you cant ping, see section 2.3 for fixing the dns issues.

## 5 Add support for Waveshare 2.13" V3 Rev 2.1 e-Ink-display

It is necessary to replace the 6 files updated/added in this [pull-request](https://github.com/evilsocket/pwnagotchi/pull/1069) to add support for the V3. Apparently the V2 is no longer produced. In case your display doesn`t work *out-of-the-box*, it is worth it to try this solution first.

hint: see if there is a "V3"-sticker on the back of the display.

*update*: you can use [this bash script](https://gist.github.com/fishd72/d3518ef40479a9272f2bd6c425b7af07) to automate the process.

### 5.1 Waveshare 2.13" V4

As of April 2024, apparently there also exists a V4 of the display. See [this pull-request](<https://github.com/evilsocket/pwnagotchi/pull/1135/commits/28c0adbb08ca5fcc6c36dc11d00f666eaecdc4cb>) for further details.

## 6 Install additional packages

Now that the internet connection has been established, it's time to install some additional packages for added functionality.

### 6.1 aircrack-ng

Install this to make the *aircrackonly* plugin work.

```bash
cd ~
sudo apt-get install aircrack-ng -y
```

if you're using default config & airackonly plugin, dont forget to add this to config:

```toml
main.plugins.aircrackonly.enabled = true
main.plugins.aircrackonly.face = "(>.<)"
```

### 6.2 hcxtools

Hcxtools are a requirement if you want to be able to make use of the *hashie.py* plugin, which can convert .pcap-files to crackable hashes.

As of april 2023, hcxtools uses libssl3 as default. This Library is currently not supported by the Kali repositories which come preloaded with the pwnagotchi. Therefore it is necessary to install the last supported release, Version 6.2.7.

```bash
cd ~
mkdir hcxtools

# install the required dependencies
apt-get install libcurl4-openssl-dev libssl-dev zlib1g-dev
```

pay attention to sudo rm -r hcxtools before, in case you tried to build the unsupported version before.

```bash
cd hcxtools
wget https://github.com/ZerBea/hcxtools/archive/refs/tags/6.2.7.zip
unzip 6.2.7.zip
```

- [ ] find out how to unzip all files in place instead of creating the 6.2.7 subfolder

Connect via sftp to move the files to the hcxtools folder.

```bash
make
sudo make install
```

make takes a while (mine took over 30 min) but should build and install  successfully.

### 6.3 pySerial

pySerial is a required dependency, if you want to use the *unofficialgps* plugin.

```bash
sudo pip3 install pySerial
```

## 7 change passwords

### 7.1 bettercap

```bash
sudo nano /etc/pwnagotchi/config.toml   
sudo nano /usr/local/share/bettercap/caplets/pwnagotchi-auto.cap
sudo nano /usr/local/share/bettercap/caplets/pwnagotchi-manual.cap
```

### 7.2 webui

```bash
sudo nano /etc/pwnagotchi/config.toml

# look for the lines
ui.web.enabled = true
ui.web.address = "0.0.0.0"
ui.web.username = "your_login_user"
ui.web.password = "your_password"
ui.web.origin = ""
ui.web.port = 8080
ui.web.on_frame = ""
```

## 8 Fix broken AI

In release 1.5.5 the AI won't start by itself. The expected behaviour for Pwnagorchi would be to start in AUTO mode and switch to AI as soon as the Neural network has been loaded.

```bash
sudo pip3 install -v --upgrade numpy
```

it taaaaakes a while, it really does. go clean your room, garage, neighbour's garage and fix your life in the meantime (2h?)

if you get a timeout error try:

```bash
sudo pip3 install --default-timeout=100 -v --upgrade numpy
```

if pwnagotchi still doesnt switch to AI after ~45min

```bash
sudo apt-get remove python-opencv
sudo apt-get install python-opencv
```

## 9 Pisugar 2 Setup

Install Pisugar Power Manager and Pisugar2 plugin. The Power Manager will set up its own neat [web service](http://10.0.0.2:8421) which allows to configure the function of the button and see some stats about your battery.

```bash
# Go to the home directory
cd ~

# Install PiSugar Power Manager 
curl http://cdn.pisugar.com/release/Pisugar-power-manager.sh | sudo bash

# Download the plugin and support library
git clone https://github.com/PiSugar/pisugar2py.git
git clone https://github.com/PiSugar/pwnagotchi-pisugar2-plugin.git

# This installs the pisugar2 package into your python library
sudo ln -s ~/pisugar2py/ /usr/local/lib/python3.7/dist-packages/pisugar2

# Installs the user-plugin
sudo ln -s ~/pwnagotchi-pisugar2-plugin/pisugar2.py /etc/pwnagotchi/custom-plugins/pisugar2.py
```

## 10 Some useful modifications

### 10.1 Set up aliases

add the followng lines to .bashrc (for pi and root separately), make sure there are no white spaces at the end!

```bash
nano ~/.bashrc
# AND/OR
sudo nano /root/.bashrc

# add these lines to the end
alias pwnlog='tail -f -n300 /var/log/pwn*.log | sed --unbuffered "s/,[[:digit:]]\{3\}\]//g" | cut -d " " -f 2-'   
alias pwnver='python3 -c "import pwnagotchi as p; print(p.version)"'

#reload bash
source ~/.bashrc
```

### 10.2 Add "SD card protection" by enabling write cache

```bash
sudo nano /etc/config.toml

fs.memory.enabled = true
fs.memory.mounts.log.enabled = true
fs.memory.mounts.data.enabled = true
```

## 11 Install additional plugins

### 11.1 Install procedure

For custom plugins to work, you have to add the path to your folder in the `config.toml`. I have added these two lines:

```bash
main.custom_plugins = "/etc/pwnagotchi/custom-plugins/"
main.custom_plugin_repos = [ "https://github.com/evilsocket/pwnagotchi-plugins-contrib/archive/master.zip",]
```

depending on where you put your *custom-plugins* folder in step 3, the procedure onwards is simple:

1. copy the *plugin_name.py* file to your folder (e.g. /etc/pwnagotchi/custom-plugins/)
2. add the required settings to the *config.toml* file. The minimum addition looks like this:

```bash
main.plugins.plugin_name.enabled = true
```

There are several community-developed plugins available, my pwnagotchi has the *exp-Plugin* by Gaelic Thunder , among others installed.

### 11.2 Example 1 - Install exp plugin

download the plugin.py file from [Github](https://github.com/GaelicThunder/Experience-Plugin-Pwnagotchi).

copy/move plugin file to `/etc/pwnagotchi/custom-plugins`.

add the following lines to `config.toml`:

```toml
main.plugins.exp.enabled = true
main.plugins.exp.lvl_x_coord = 0
main.plugins.exp.lvl_y_coord = 93
main.plugins.exp.exp_x_coord = 38
main.plugins.exp.exp_y_coord = 93
main.plugins.exp.bar_symbols_count = 12
```

### 11.3 Example 2 - Install hashie plugin

download hashie.py from [official git](https://github.com/evilsocket/pwnagotchi-plugins-contrib/blob/master/hashie.py).

copy hashie.py to custom-plugins (see above).

add line in config.toml:

```toml
main.plugins.hashie.enabled = true
```

## 12 Housekeeping

### 12.1 update small pwnagotchi face (result from Waveshare 3 files)

source: <https://www.reddit.com/r/pwnagotchi/comments/u4q18m/how_to_fix_the_small_face_issue_with_waveshare_v3/>
change face size in
..pwnagotchi/ui/hw/waveshare3.py, Line 13

## 13 Additional Tips & Tricks

### 13.1 default folder locations

|folder|location|
|---|---|
|default plugins directory|/usr/local/lib/python3.7/dist-packages/pwnagotchi/plugins/default/|
|custom plugins directory|/etc/pwnagotchi/custom-plugins/|
|config directory|/etc/pwnagotchi/
|neural network|/root/brain.nn|
|information about nn|/root/brain.json|
|Logs|/var/log/pwnagotchi.log|
|Memory|/root/peers/|
|handshakes|/root/handshakes/|

### 13.2 example for a display layout

<https://www.reddit.com/r/pwnagotchi/comments/pn6ztt/pwnagotcha_running_custom_scripts_including_exp/>

### 13.3 other plugins/fixes

- change pwnagotchi at /usr/local/bin (home_base epoch cycling fix)
<https://github.com/evilsocket/pwnagotchi/pull/1003>

- change  webgpsmap.html /usr/local/lib/python3.7/dist-packages/pwnagotchi/plugins/default for custom made ver

- change watchdog /usr/local/lib/python3.7/dist-packages/pwnagotchi/plugins/default to v1.0.0
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/watchdog.py>

- change paw-gps to 1.0.1
<https://github.com/evilsocket/pwnagotchi/pull/1054>

- change clock.py 1.0.3  ##changes allow you to customise the displayed time format by editing config.toml, e.g. switching from 12 to 24 hours clock format, or adding seconds
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/29/files>

- change hashie.py 1.0.3  ##newer than <https://github.com/PwnPeter/pwnagotchi-plugins>  hashie 1.0.3 //it migh be slower than pwnpeter's version//do more tests//
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/36/files>

- display_version.py 1.0.0
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/35/files>

- wpa-sec-list 1.0.0   ##List cracked passwords from wpa-sec
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/34/files>

- HandshakesDL 0.2.2 ##Download handshake captures from web-ui [newer than handshakes 1.0.3 https://github.com/PwnPeter/pwnagotchi-plugins]
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/27/files>

- aircrack-ng 2.0.0.
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/aircrackonly.py>

- webcfg update?
<https://github.com/dadav/pwnagotchi/blob/master/pwnagotchi/plugins/default/webcfg.py>

- home_base 1.0.0 & away_base 1.0.0
<https://github.com/troystauffer/home_base>
plus this fix
<https://github.com/evilsocket/pwnagotchi/pull/1003>

- unoficialGPS 1.0.0
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/pull/22/files>

- auto_update.py 2.0.0
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/auto_backup.py>
update it with this:
<https://github.com/evilsocket/pwnagotchi-plugins-contrib/issues/30>

- exp 1.0.5
<https://github.com/GaelicThunder/Experience-Plugin-Pwnagotchi>

- onlinehashcrack 2.1.5
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/onlinehashcrack.py>

- wpa-sec.py 3.0.2
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/wpa-sec.py>

- screen_refresh 2.0.0
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/screen_refresh.py>

- wpa-sec2  #It will only do a check and make .gps.cracked files from the downloaded .cracked.potfile after it downloads from wpa-sec
<https://github.com/xenDE/pwnagotchi-tools/issues/1>
   #yml config for it below. NOTE! i didnt test this plugin yet, you need to translate it to toml before you put it in a config
          wpa-sec2:
            enabled: false
            api_key: ''
            api_url: "https://wpa-sec.stanev.org"
            download_results: true

- watchdog.py 1.0.0
<https://github.com/dadav/pwnagotchi-custom-plugins/blob/master/watchdog.py>

- paw-gps.py 1.0.1
<https://github.com/evilsocket/pwnagotchi/pull/1054>

## 14 known bugs

- bettercap not showing any signals
- activation of *onlinehashcrack* or *wpa-sec* plugin leads to pwnagotchi not starting up

## To Do

- [x] add to preface the used hardware, including revisions
- [x] explain details of Windows Connection Script and Ethernet Settings
- [x] copy all content from original guide over to this document
- [ ] add references and links to jump around in this document
- [ ] clean up
