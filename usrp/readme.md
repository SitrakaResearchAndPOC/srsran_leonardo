# SRSRAN LEONARDO USRP
## I. Installing tools
```
rm -rf srsran_leonardo_usrp ; mkdir srsran_leonardo_usrp && cd srsran_leonardo_usrp
```
```
apt update
```
```
apt install docker.io wget unzip
```
```
apt install linux-tools-common linux-tools-generic
```
```
cpupower frequency-set -g performance
```

## II. Download Dockerfile
```
[ -f Dockerfile.usrp ] && rm -rf Dockerfile.usrp ; \
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/srsran_leonardo/refs/heads/main/usrp/configs/Dockerfile.usrp
```
## III. Building images
```
docker  build -t srsran_leonardo_usrp:v1 -f Dockerfile.usrp .
```

## IV. Running images
```
docker rm -f srsran_leonardo_usrp 2> /dev/null ; \
docker run -tid --privileged \
  --cgroupns=host \
  --net=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -v /dev:/dev \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /tmp/.X11-unix:/tmp/.X11-unix:ro \
  -v /home/user/.Xauthority:/home/user/.Xauthority:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --env="DISPLAY=$DISPLAY" \
  --env="LC_ALL=C.UTF-8" \
  --env="LANG=C.UTF-8" \
  --env="NAME_PLUTO=pluto" \
  --cap-add=sys_nice \
  --cap-add=ipc_lock \
  --ulimit rtprio=99 \
  --ulimit memlock=-1 \
  --volume /run/dbus/system_bus_socket:/run/dbus/system_bus_socket \
  --volume /run/avahi-daemon/socket:/run/avahi-daemon/socket \
  --name srsran_leonardo_usrp \
  --hostname srsran_leonardo_usrp \
  srsran_leonardo_usrp:v1
```

## V. Testing driver UHD
```
xhost +
```
```
docker exec -ti srsran_leonardo_usrp bash -c 'uhd_find_devices'
```
```
docker exec -ti srsran_leonardo_usrp bash -c 'uhd_usrp_probe'
```
Finding APN : 
```
docker exec -ti srsran_leonardo_usrp bash -c 'cat /root/.config/srsran/epc.conf | grep apn'
```
verifying user : 
```
docker exec -ti srsran_leonardo_usrp bash -c 'cat  /root/.config/srsran/user_db.csv | grep ue'
```

## VI. Running srsRAN LTE

### ON TERMINAL 1
```
cpupower frequency-set -g performance && docker exec -ti  srsran_leonardo_usrp bash -c 'srsepc'
```
Tape ctrl+shift+T

### ON TERMINAL 2
```
cpupower frequency-set -g performance && docker exec -ti  srsran_leonardo_usrp bash -c 'srsenb'
```
## VII. Sharing Internet
New terminal , tape ctrl+shit+t </br>
### Finding interface which gives internet
```
ifconfig
```
Let name the interface <if_name> 
### sharing lte traffic
```
command -v wget >/dev/null 2>&1 || \
(apt-get update && apt-get install -y wget ) ; \
[ -f srsepc_if_masq.sh ] || \
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/plutosdr_srsran/refs/heads/main/srsepc_if_masq.sh && chmod +x  *.sh
```
```
bash srsepc_if_masq.sh <if_name>
```
### Routing internet over the interface
```
sysctl -w net.ipv4.ip_forward=1
```
### Disabling firewall 
```
ufw disable
```

## VIII. Configuring SIM & APN
1) Download [GR-SIM](https://github.com/SitrakaResearchAndPOC/gr-sim-write) </br>
2) Download grsp config [CONFIG](https://github.com/SitrakaResearchAndPOC/srsran_leonardo/blob/main/configs_simcard_grsp/srsran_leonardo_xor.grsp) ; my SIM support only xor for authentication algorithm </br>
3) Load and write the config on the green card using card write </br>
4) Plug and search network on parameter/sim_card/search_network and select network mcc=208 and mnc=92 </br>
5) If the simcard is authenticated on the network for future searching it's more quick to active/desactivate avion mode </br>
6) Configure APN indeed name should be `srsapn` and mcc on the apn should be 208 and mnc should be 92 </br>
