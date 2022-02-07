# RaspberryNas

Simple Network attached Storage only working in local network environment. The terminal commands were executed on Ubuntu LTS 20.04.
Will be extended in the future for external accessibility... 

## Requirements 

### Physical components:
- Raspberry Pi 4
- External Harddrive (USB)

### Software:
- OpenMediaVault: https://www.openmediavault.org/
- NextCloud: https://nextcloud.com/
- Docker: https://www.docker.com/


## Basic Installation

1. Plug in HDMI-cable, network-cable, keyboard and powercable
2. Boot SD card and install 32bit RasbianOS without Desktop support https://www.raspberrypi.com/software/operating-systems/ (maybe preinstall with Noob)
3. Log in with username `pi` and password `raspberry`
4. First update the system with: `sudo apt-get update && sudo apt-get upgrade`
5. Change password with `passwd`
6. Enable ssh support: `sudo raspi-config` and select `Interface Options`
7. Make IP static such that the DHCP server will not change it dynamically (Can also be done in the routers configuration) in Steps 8-17:
8. Find out local IP by typing: `hostname -l`
9. Let’s first retrieve the currently defined router for your network by running the following command: `ip r | grep default`
10. Make note of the first IP metioned in the output
11. Next, let us also retrieve the current DNS server by typing: `sudo nano /etc/resolv.conf`
12. In the file you can see the nameserver's configured IP
13. Now create a new file with: `sudo nano /etc/dhcpcd.conf`
14.  Copy paste the following content into the file. 
```
interface eth0
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP>
```
15. Fill in the IP from step 8 into `<STATICIP>` and the IP from step 9/10 into `<ROUTERIP>` and finally the IP form step 12 into `<DNSIP>`
16. Now save the file by pressing `CTRL + X` then `Y` followed by `ENTER`
17. Run `sudo reboot`
18. Connect via ssh to the Raspberry Pi instance by opening a terminal on another pc and type in following command by using the IP from step 8: `sudo ssh pi@<PI's static IP ADDRESS>`
19. Type in new password from step 5
20. Now you should be connected to the Raspberry instance

## OpenMediaVault Installation
  
  1. Install OpenMediaVault by copying this command into running ssh instance: `sudo wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash`
  2. Check for more installation info on: `https://github.com/OpenMediaVault-Plugin-Developers/installScript`
  3. Open a new browser window and type in the Raspberry Pi IP Address
  4. Log into promt: Username: `admin` Password: `openmediavault`
  5. Plug your External USB drive into your Raspberry Pi
  6. Go to the `Storage` Tab an check if the device has been detected
  7. Now change to the `File Systems` Tab and hit create. Now The currect path of the drive must be selected. The Fileformat must be `EXT4`
  8. Now go to `Shared Folder` and hit create. Name your folder as you like and select the file system that you selected in step 7. Leave the permissions and relative path as it is. 
  9. Select the created folder and check the privileges such that the user `pi` has read and write access
  10. Remember the absolute path because we need this later in the nextcloud setup

## Nexcloud Installation (Docker)
1. Install docker by: `curl -sSL https://get.docker.com | sh`
2. Allow `pi` user to run docker with: `sudo usermod -aG docker ${USER}`
3. Install necessary packages for docker-compose in steps 4-7:
4. `sudo apt-get install libffi-dev libssl-dev`
5. `sudo apt install python3-dev`
6. `sudo apt-get install -y python3 python3-pip`
7. `sudo pip3 install docker-compose`
8. Configure that docker will always restart after booting up the Raspberry Pi by: `sudo systemctl enable docker`
9. Copy the docker-compose.yml in this Repo to the Raspberry Pi instance and fill in the necessary passwords for the database and the user
10. Now copy the absolute path (Step 10 from the OpenMediaVault installation) to the position `volumes` such that nextcloud will copy all files to the external harddrive
11. Open a new browser window and type in the Raspberry Pi IP Address with port 8080 and create a new user
12. Nextcloud will install the necessary applications and your cloud storage is now set up!
