# RaspberryNas

Simple Network attached Storage accessible from outside local network. The terminal commands were executed on Ubuntu LTS 20.04.

## Requirements 

### Physical components:
- Raspberry Pi 4
- External Harddrive (USB)

### Software:
- OpenMediaVault: https://www.openmediavault.org/
- NextCloud: https://nextcloud.com/
- Docker: https://www.docker.com/
- Nginx Proxy Manager: https://nginxproxymanager.com/


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
  4. Log into prompt: Username: `admin` Password: `openmediavault`
  5. Plug your External USB drive into your Raspberry Pi
  6. Go to the `Storage` Tab an check if the device has been detected
  7. Now change to the `File Systems` Tab and hit create. Now The currect path of the drive must be selected. The Fileformat must be `EXT4`
  8. Now go to `Shared Folder` and hit create. Name your folder as you like and select the file system that you selected in step 7. Leave the permissions and relative path as it is. 
  9. Select the created folder and check the privileges such that the user `pi` has read and write access
  10. Remember the absolute path because we need this later in the nextcloud setup
  11. (Optional) Additionally Portainer can be installed in the Openmediavault GUI running on port 80 per default. This allows easier docker container management

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
11. Execute `sudo docker-compose up -d` in the same directory where you saved the docker compose file, the nextcloud server and the database will be installed
12. Open a new browser window and type in the Raspberry Pi IP Address with port 8080 and create a new user
13. Nextcloud will install the necessary applications and your cloud storage is now set up!

## Make Nextcloud accessible from outside private network
This optional step makes it easier to manage your NAS on your mobile phone from anywhere around the world.

### IPv4 to IPv6 mapper with virtual server
The salt box of my ISP does not assign static IPv4 addresses, so it cannot be used to address the RaspberryPi 4 outside the local network. But the Ipv6 addresses are assigned statically. Therefore, a virtual server was set up at Netcup: https://www.netcup.de, which has a static IPv4 address and through which all requests to nextcloud should run. The virtual server receives a request via IPV4 and maps it to the static IPV6 address of the Raspberry Pi. This address can be looked up at the local router. To achieve this, the following commands must be entered on the virtual server:
1. Install firewall software with `sudo apt install ufw`
2. Configure ports 22, 80 and 443: 
```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
```
3. Activate firewall changes: `sudo ufw enable`
4. Install 6tunnel mapping software: `sudo apt-get install 6tunnel`
5. Type following commands to create a IPv6 to IPv4 mapping for port 80 and 443, the xxxx.. placeholder is Raspberry Pi's static Ipv6 address: 
```
6tunnel -4 80 xxxx:xxxx:xxxx:xxxx:xxxx:xxxx 80
6tunnel -4 443 xxxx:xxxx:xxxx:xxxx:xxxx:xxxx 443
```
6.Check the config with: `ps -aux | grep 6tunnel`
7. Optional: Make sure the commands are executed each time the server restarts: `crontab -e` and add the commands with `@reboot` as prefix.
8. Copy the used static IPv4 address from the virtual server

### Domain name for accesing Nextcloud
1. Purchase a domain e.g. from namecheap: https://www.namecheap.com
2. Define a subdomain e.g. `nextcloud.yourdomain.com` and map the IPv4 address from the virtual server to it

### Reverse Proxy with Nginx
We want to hide the requests inside our network from the outside world, this can be done with a reversed proxy. Make sure that you forward all requests on port 80 and 443 are redirected to the proxy, this can be done in the router settings.
1. SSH into Raspberry Pi
2. Create new folder for nginx: `mkdir nginx` and move into it: `cd nginx`
3. Create config file: `nano config.json`
4. Add following content and change user and password accordingly:

```
{
  "database": {
    "engine": "mysql",
    "host": "db",
    "name": "npm",
    "user": "changeme",
    "password": "changeme",
    "port": 3306
  }
}
```
5. Now copy the content of nginx/docker-compose.yml into the directory and change name and passwords accordingly.
6. Start nginx proxy manager: `docker-compose up -d`
7. Open up the GUI on port 81 and log in with:
```
Username: admin@example.com
Password: changeme
```
8. Click on `Add Proxy Host` and enter the defined subdomain for the nextcloud server and also Raspberry Pi's static Ipv4 address and the specified port on which the nextcloud server is listening. Make sure `Block Common Exploits` is check
9. Click on the `SSL` tab and choose the `Request a new SSL certificate` option and enter the same subdomain again and make sure `Force SSL` and `HTTP/2 Support` is checked (must be controlled again after saving) and save it.

### Nextcloud config
In order to allow other domains to access the nextcloud server the config.php from nextcloud must be modified to overwrite the protocols to https and add our defined subdomain to the trusted domains.

```
'overwriteprotocol' => 'https',
'trusted_domains' =>
  array (
    0 => '192.168.0.29',
    1 => 'cloud.example.com',
  ),
```
The `config.php` file is located at `/var/www/html/config/config.php` and no editor is installed to modify it. A simple solution is to copy the file to the local machine by `docker cp NEXTCLOUD_CONTAINER_NAME:/var/www/html/config/config.php config.php` and then modify it and copy it back into the nextcloud container. Make sure to change the owner of the file such that we do no have file access problems. For that go into the container with the command
`docker exec -it 'NEXTCLOUD_CONTAINER_NAME' /bin/bash` and change the owner and the group by `chown www-data:root config.php`.


Now nexcloud should be accesible over https with your specified domain!!










