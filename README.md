# About
This project is a guide and files to convert an old eeePC to an OctoPrint station in order to manage my 3D printer in my workshop.

# Disclaimer
English is not my native language, and I did not spend a lot of time writing beautiful sentences for this guide.

The important thing for me when writing this guide was to record all the steps that I made to achieve my goal of using this old EeePC as an OctoPrint server and "local station".

# Prerequisites
## Skills
I'm very familiar with Linux OS, and I will use command line tools because I'm more efficient using a keyboard than a mouse.
So most instructions will be terminal commands.

## Hardware
I have an old Asus EeePC T101MT, with a touchscreen. The battery is dead, but everything else is working fine (touchscreen, keyboard, touchpad, network...). As I'm not using it anymore, it would be great to give it a second life, by making it a control station for my 3D Printer (ANET A8).

(Also confirmed to work with the EeePC 1005-HA (no touchscreen) and the Monoprice Select Mini v2.)

This will let me control the printer when I'm close to it, with the EeePC touchscreen, and also when I'm away, via network.

And to be perfect, adding a webcam will be great to check regularly if printing is not going wrong.

## Software
### OS
The EeePC is old and not a powerful computer, so I will use a lightweight Linux distribution, like [Lubuntu](https://lubuntu.net/).

Later I'll maybe try Ubuntu Server, add an X server with no window manager, and start only the tool to control the printer (via [Chromium in kiosk mode](https://www.sylvaindurand.fr/launch-chromium-in-kiosk-mode/) for example) if Lubuntu performs unsatisfactory.

### Printer control
I'll use [OctoPrint](https://octoprint.org) because I can use it locally (displaying it in a web browser on the EeePC) and remotely (on my desktop computer, or smartphone, even when I'm not at home).

# Installation
## OS
So, I decided to try Lubuntu Linux. Doing this I could use the EeePC as a little computer in my workshop when need to search things on the Internet. But by default, it will always display the OctoPrint tool.

Go to the [Lunbuntu website](https://lubuntu.net),  [download Lubuntu](https://lubuntu.net/downloads/), and install it following [official docs](https://docs.lubuntu.net/lubuntu_installation).

Once this is done, you have a functional Linux system, and a user, with a graphical interface.
Connect with your user credentials (created during installation), open a terminal and update your OS:
```
sudo apt update
sudo apt upgrade
```
## OctoPrint
This section will describe all steps to install OctoPrint and make it always open on the screen.

This solution is based on the [OctoPrint Raspbian installation guide](https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspbian/2337)

### Overview
In short, we will perform these tasks:
0. Make the Lunbuntu accessible via SSH for remote administration
1. Create a sytem user that will run the OctoPrint service
2. Install all Python packages needed to perform the OctoPrint installation
3. Install OctoPrint
4. Configure a service to run OctoPrint as a daemon
5. Add a proxy to make OctoPrint accessible via simple URLs (without the port 5000 info)
6. Configure `mjpg-streamer` to add a webcam to OctoPrint
7. Make OctoPrint open automatically at EeePC startup (and always be open, even if closed accidentally).
8. Give OctoPrint the rights to restart the system
9. Add a touchUI plugin, to make OctoPrint more usable on the little screen of the EeePC

OK, now time for action!

### Add SSH Access
Simply install the OpenSSH Server package in Lubuntu.

In a terminal:
```
sudo apt install openssh-server
```
This allows the remainder of this guide to be followed remotely, should you choose to.

### Create OctoPrint System User
Then, we need to create a system user that will be used to run OctoPrint as a daemon or system service.

In a terminal:
```
sudo groupadd -r octoprint
sudo useradd -r -g octoprint -d "/var/octoprint" -s "/bin/bash" octoprint
sudo mkdir -p /var/octoprint
sudo usermod -a -G tty octoprint
sudo usermod -a -G dialout octoprint
sudo usermod -a -G video octoprint
sudo chown octoprint:octoprint -R /var/octoprint
```

*TODO: add a detailed description of each command, what it does and what is it for*

### Install Prerequisite Python Packages
This step is to install all Python packages needed to perform the OctoPrint installation.

In a terminal:
```
sudo apt install python3 python3-pip python3-dev python3-setuptools python3-venv git libyaml-dev build-essential libffi-dev libssl-dev
```

### Install OctoPrint
A simple but important step, this is the OctoPrint installation:

```
sudo su octoprint
cd /var/octoprint
mkdir octoprint-venv
virtualenv octoprint-venv
source octoprint-venv/bin/activate
pip install pip --upgrade
pip install octoprint
deactivate
```

Verify that OctoPrint is correctly installed and can run:
```
./octoprint-venv/bin/octoprint serve
```

Running the above command should result in a lot of output. Wait until the output stops displaying new lines, and open a web browser to visit http://localhost:5000 (if performing these steps over SSH, substitute `localhost` with the EeePCs IP). The OctoPrint configuration screen should be displayed.

*TODO : add a screenshot*

If everything is as expected, stop the OctoPrint server using `Ctrl-C` and return to your user from the octoprint user:
```
exit
cd ~
```

### Run OctoPrint as a Daemon
In this section we will install and configure a service to run OctoPrint as a system daemon. This ensures it will always be running, even after a system restart.

Service scripts are available at [OctoPrint github page](https://github.com/foosel/OctoPrint).

#### Option A: Init Scripts
```
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init
sudo mv octoprint.init /etc/init.d/octoprint
sudo chmod +x /etc/init.d/octoprint
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default
sudo mv octoprint.default /etc/default/octoprint
```

Then we need to configure the service. Open `/etc/default/octoprint` with your prefered text editor, and change configuration to match this:

```
# Configuration for /etc/init.d/octoprint

# The init.d script will only run if this variable non-empty.
OCTOPRINT_USER=octoprint

# base directory to use
BASEDIR=/var/octoprint/.octoprint

# configuration file to use
CONFIGFILE=/var/octoprint/.octoprint/config.yaml

# On what port to run daemon, default is 5000
PORT=5000

# Path to the OctoPrint executable, you need to set this to match your installation!
DAEMON=/var/octoprint/octoprint-venv/bin/octoprint

# What arguments to pass to octoprint, usually no need to touch this
DAEMON_ARGS="--port=$PORT"

# Umask of files octoprint generates, Change this to 000 if running octoprint as its own, separate user
UMASK=022

# Process priority, 0 here will result in a priority 20 process.
# -2 ensures Octoprint has a slight priority over user processes.
NICELEVEL=-2

# Should we run at startup?
START=yes
```

Changes are made to: OCTOPRINT_USER, BASEDIR, CONFIGFILE, and DAEMON.

Then add the script to default runlevel with:
```
sudo update-rc.d octoprint defaults
```
#### Option B: SystemD Service
In your home directory, fetch the systemd service file:
```
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.service
```
Edit the file to match:
```
[Unit]
Description=The snappy web interface for your 3D printer
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=octoprint
ExecStart=/var/octoprint/octoprint-vdev/bin/octoprint
Nice=-2

[Install]
WantedBy=multi-user.target
```
`ExecStart` was changed, and `Nice` was added.

Next, run the following:
```
sudo mv octoprint.service /etc/systemd/system/octoprint.service
sudo chmod +x /etc/systemd/system/octoprint.service
sudo systemctl daemon-reload
```
#### Run OctoPrint
And finally, you can start/stop/restart the OctoPrint daemon using command:
```
sudo service octoprint {start|stop|restart}
```

For now, start the octoprint service with: `sudo service octoprint start` and check that it is running by opening http://localhost:5000 in a web browser (again, substituting `localhost` with the EeePC's IP if performing these steps remotely).

### Make OctoPrint Accessible via Simple URLs
As the `octoprint` user is not at root level, it cannot make the OctoPrint server to listen on port 80.

So, to make OctoPrint accessible via the normal HTTP port (80), we need to install a reverse proxy. We will use HAProxy as it is simple to configure.

In a terminal:
```
sudo apt install haproxy
```

Then we need to configure HAproxy. Open `/etc/haproxy/haproxy.cfg` and add three sections at the end (after general and defaults): `frontend public`, `backend octoprint`, `backend webcam`. (this last section is needed if you add a a webcam to OctoPrint, as described later in this guide)

```
frontend pfrontend
        bind :::80 v4v6
        use_backend webcam if { path_beg /webcam/ }
        default_backend octoprint

backend octoprint
        reqrep ^([^\ :]*)\ /(.*)     \1\ /\2
        option forwardfor
        server octoprint1 127.0.0.1:5000

backend webcam
        reqrep ^([^\ :]*)\ /webcam/(.*)     \1\ /\2
        server webcam1  127.0.0.1:8080
```

Then restart the HAProxy service:
```
sudo service haproxy restart
```

And check if OctoPrint is accessible by visiting http://localhost in a web browser.

One last thing to do is to restrict OctoPrint to bind only to the loopback interface, for security reasons. Open the file `/var/octoprint/.octoprint/config.yaml` and add this line in the `server` section:
```
    host: 127.0.0.1
```

Then restart octoprint service `sudo service octoprint restart`.

### Plug Printer and Webcam
It is time to connect the 3D printer to the EeePC and add a webcam.
I plugged them using USB.
I checked the webcam is workging with Cheese, a small tool to test webcams.

### Add a webcam to OctoPrint
To be able to stream images from printing process, we need to install the `mjpg_streamer` tool.

This is done with those commands:
```
sudo apt install cmake libjpeg9-dev libjpeg-turbo8-dev ffmpeg
sudo su octoprint
cd ~
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
make
```
To test if mjpg-streamer works with your webcam, run this command:
```
./mjpg_streamer -i "./input_uvc.so -y -d /dev/video2" -o "./output_http.so -w ./www"
```
Note: change `/dev/video2` to the one corresponding to your webcam.

Then open a browser localy and go to url http://localhost:8080; you should see a web page with a lot of information about `mjpg_streamer`, and a tab named "Stream" in wich you should see the output of your webcam.

Stop the stream using `Ctrl-C`.

Now, we will add scripts to be able to start and stop webcam from OctoPrint.

Create script directory:
```
mkdir ~/scripts
```
then create the file `~/scripts/webcam` and insert its content:
```
#!/bin/bash
# Start / stop streamer daemon

case "$1" in
    start)
        /var/octoprint/scripts/webcamDaemon >/dev/null 2>&1 &
        echo "$0: started"
        ;;
    stop)
        pkill -x webcamDaemon
        pkill -x mjpg_streamer
        echo "$0: stopped"
        ;;
    *)
        echo "Usage: $0 {start|stop}" >&2
        ;;
esac
```

Create another file: `~/scripts/webcamDaemon` and add its content:
```
#!/bin/bash

MJPGSTREAMER_HOME=/var/octoprint/mjpg-streamer/mjpg-streamer-experimental
MJPGSTREAMER_INPUT_USB="input_uvc.so"
MJPGSTREAMER_INPUT_RASPICAM="input_raspicam.so"

# init configuration
camera="usb" #auto, usb, raspi
camera_device="/dev/video2" #/dev/video0, /dev/video1, ...
camera_usb_options="-y -r 640x480 -f 10 -d $camera_device"
camera_raspi_options="-fps 10"

if [ -e "/boot/octopi.txt" ]; then
    source "/boot/octopi.txt"
fi

# runs MJPG Streamer, using the provided input plugin + configuration
function runMjpgStreamer {
    input=$1
    pushd $MJPGSTREAMER_HOME
    echo Running ./mjpg_streamer -o "output_http.so -l 127.0.0.1 -w ./www" -i "$input"
    LD_LIBRARY_PATH=. ./mjpg_streamer -o "output_http.so -w ./www" -i "$input"
    popd
}

# starts up the RasPiCam
function startRaspi {
    logger "Starting Raspberry Pi camera"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_RASPICAM $camera_raspi_options"
}

# starts up the USB webcam
function startUsb {
    logger "Starting USB webcam"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_USB $camera_usb_options"
}

# we need this to prevent the later calls to vcgencmd from blocking
# I have no idea why, but that's how it is...
vcgencmd version

# echo configuration
echo camera: $camera
echo usb options: $camera_usb_options
echo raspi options: $camera_raspi_options

# keep mjpg streamer running if some camera is attached
while true; do
    if [ -e "$camera_device" ] && { [ "$camera" = "auto" ] || [ "$camera" = "usb" ] ; }; then
        startUsb
    elif [ "`vcgencmd get_camera`" = "supported=1 detected=1" ] && { [ "$camera" = "auto" ] || [ "$camera" = "raspi" ] ; }; then
        startRaspi
    fi

    sleep 120
done
```
**Important: in this script, you must adapt configuration for your needs: camera, camera_raspi_options, camera_usb_options.**

Make both files executable by only user `octoprint`:
```
chmod u+x ~/scripts/*
```

Then, to be able to start and stop the streaming from OctoPrint, we need to add  actions to `~/.octoprint/config.yaml`:
```
system:
  actions:
   - action: streamon
     command: /var/octoprint/scripts/webcam start
     confirm: false
     name: Start video stream
   - action: streamoff
     command: /var/octoprint/scripts/webcam stop
     confirm: false
     name: Stop video stream
   - action: printeron
     command: curl "http://<yourJeedomURL>/core/api/jeeApi.php?apikey=<yourAPIKey>&type=cmd&id=<yourCmdID>"
     confirm: false
     name: Turn on Anet A8
   - action: printeroff
     command: curl "http://<yourJeedomURL>/core/api/jeeApi.php?apikey=<yourAPIKey>&type=cmd&id=<yourCmdID>"
     confirm: "ATTENTION ! Vous allez éteindre l'imprimante 3D !"
     name: Turn off Anet A8
```

Finally, logout from the `octoprint` user (`exit`).

### Configure OctoPrint
Now, it is time to configure OctoPrint by following the Setup Wizard displayed when you go to OctoPrint URL.

I decided to do this remotely from my desktop computer.

For more information how to setup OctoPrint, please refer to the official documentation.

Some settings to setup in the wizard:
* Server commands section
  * Restart OctoPrint: `sudo service octoprint restart`
  * Restart system: `sudo shutdown -r now`
  * Shutdown system: `sudo shutdown -h now`
* Webcam & Timelapse section
  * Stream URL: `/webcam/?action=stream`
  * Snapshot URL: `http://127.0.0.1:8080/?action=snapshot`
  * Path to FFMPEG: `/usr/bin/ffmpeg`


### make OctoPrint to be displayed automatically at EeePC startup
I created a new user (`impression3D`) using Lubuntu system tools, I made the taskbar "retractable", and disabled desktop icons.

Then create this little script (named `/home/impression3D/.bin/start_firefox.sh`) that will launch Firefox and put it in fullscreen mode:
```
#!/bin/bash
firefox -url http://localhost/ &
xdotool search --sync --onlyvisible --class "Firefox" windowactivate key F11
```

You need to install `xdotool` for the script to run, and make the script executable:
```
sudo apt install xdotool
chmod u+x ~/.bin/start_firefox.sh
```

Finally, add this script in the autostart configuration of Lubuntu (using system config tools).

### Give OctoPrint the Rights to Restart the System and to Restart Itself
By default, the `octoprint` system user does not have permission to use the `sudo` command. And even if it could, it does not have a password, so it would not be able to perform credential validation.

The solution is to give the `octoprint` user permission to run only the needed commands using `sudo` without entering a password.

This is done by adding a configuration file for sudoers: `/etc/sudoers.d/octoprint-shutdown`.

So create that file and add this content:
```
octoprint ALL=NOPASSWD: /bin/shutdown, /usr/sbin/service
```

### Add the TouchUI Plugin to Make OctoPrint More Usable on the Little Screen of the EeePC
To do so, simply install the TouchUI plugin from the OctoPrint system plugin menu.

# TODO
* add SSL to HAProxy configuration in order to secure network communication from clients to OctoPrint.
* redo installation without window manager (Ubuntu Server + X and TouchUI autostart: https://github.com/BillyBlaze/OctoPrint-TouchUI-autostart)
