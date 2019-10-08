# About
This project is a guide and files to convert an old eeePC to an OctoPrint station in order to manage my 3D printer in my workshop.

# Prerequities
## Skills
I'm very familiar with Linux OS, and I will use command line tools because I'm more efficient using a keyboard than a mouse.
So most of instructions will be described using a terminal.

## Hardware
I have an old Asus EeePC T101MT, with a touchscreen. The battery is dead, but everything else is working fine (touchscreen, keyboard, touchpad, network...). As I'm not using it anymore, it would be great to give it a second life, by making it a control station for my 3D Printer (ANET A8).

This will let me control the printer when I'm close to it, with the EeePC touchscreen, and also when I'm away, via network.

And to be perfect, adding a webcam will be great to check regularly if printing is not going wrong.

## Software
### OS
The EeePC is an old and not powerfull computer, so I will use a small linux distribution, like [Lubuntu](https://lubuntu.net/).

Later I'll maybe try an ubuntu server, add a X server with no window manager, and start only the tool to control the printer (via [chromium in kiosk mode](https://www.sylvaindurand.fr/launch-chromium-in-kiosk-mode/) for example) if the Lubuntu don't feed my needs.

### Printer control
I'll use [OctoPrint](https://octoprint.org) because I can use it localy (displaying it in a webbrowser on the EeePC) and remotely (on my desktop computer, or smartphone, even when I'm not at home).

# Installation
## OS
So, I decided to try a Lubuntu linux. Doing this I could use the EeePC as a little computer in my workshop when needed to search things on the Internet. But by default, it will always display the OctoPrint tool.

Go to [Lunbuntu website](https://lubuntu.net),  [download Lubuntu](https://lubuntu.net/downloads/), and install it following [official docs](https://docs.lubuntu.net/lubuntu_installation).

Once this is done, you have a plenty fonctionnal linux system, and a user, with a graphical interface.
Connect with your user credentials (created during installation), open a terminal and update your OS :
```sudo apt update
sudo apt upgrade

## OctoPrint
### System user
### Python
### OctoPrint
### 
