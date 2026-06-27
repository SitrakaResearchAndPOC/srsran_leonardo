# SRSRAN LEONARDO BLADERF
## I. Installing tools
```
rm -rf srsran_leonardo_bladerf ; mkdir srsran_leonardo_bladerf && cd srsran_leonardo_bladerf
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
[ -f Dockerfile.bladerf ] && rm -rf Dockerfile.bladerf ; \
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/srsran_leonardo/refs/heads/main/bladerf/configs/Dockerfile.bladerf
```
## III. Building images
```
docker  build -t srsran_leonardo_bladerf:v1 -f Dockerfile.bladerf .
```

## IV. Running images
```
docker rm -f srsran_leonardo_bladerf 2> /dev/null ; \
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
  --name srsran_leonardo_bladerf \
  --hostname srsran_leonardo_bladerf \
  srsran_leonardo_bladerf:v1
```

## V. Testing driver BladeRF
```
xhost +
```
Verify connexion of BladeRF
```
docker exec -ti srsran_leonardo_bladerf bash -c "bladeRF-cli -p"
```
Verify information of BladeRF
```
docker exec -ti srsran_leonardo_bladerf bash -c "echo 'info' | bladeRF-cli -i"
```
Verify version of BladeRF
```
docker exec -ti srsran_leonardo_bladerf bash -c "echo 'version' | bladeRF-cli -i"
```
eg of output :  
```
root@dragon:/home/dragon# docker exec -ti srsran_leonardo_bladerf bash -c "echo 'version' | bladeRF-cli -i"  
[WARNING @ host/libraries/libbladeRF/src/board/bladerf2/bladerf2.c:360] Using legacy message size. Consider upgrading firmware >= v2.5.0 and fpga >= v0.16.0 

  bladeRF-cli version:        1.10.0-git-41b7fc7-dirty 
  libbladeRF version:         2.6.1-git-41b7fc7-dirty 

  Firmware version:           2.4.0-git-a3d5c55f 
  FPGA version:               0.14.0 (configured by USB host)
```
</br>   
Version of libbladerf >= version firmware and should without warning </br>
find the adequat version at  https://www.nuand.com/fx eg : 2.6.0 </br>

```
docker exec -ti srsran_leonardo_bladerf bash -c "wget https://www.nuand.com/fx3/bladeRF_fw_v2.6.0.img"
```
FLASHING THE FIRMEWARE : Could break bladerf if the action is suddenly stopped
```
docker exec -ti srsran_leonardo_bladerf bash -c "bladeRF-cli -f bladeRF_fw_v2.6.0.img"
```
Unplug the bladerf wait 10second and replug </br>
find adequat verstion at https://www.nuand.com/fpga_images/ ; </br>
the date of the fpga should be <= date of the firmeware </br>
eg date of firmware : 2025-05-11 - v2.6.0 and date of fpga 2025-05-11 - fpga_v0.16.0 </br>
```
docker exec -ti srsran_leonardo_bladerf bash -c "wget https://www.nuand.com/fpga/v0.16.0/hostedxA4.rbf"
```
```
docker exec -ti srsran_leonardo_bladerf bash -c "bladeRF-cli -l hostedxA4.rbf"
```
```
docker exec -ti srsran_leonardo_bladerf bash -c "echo 'version' | bladeRF-cli -i"
```
The display of version should be without @warning 


## VI. Running srsRAN LTE

### ON TERMINAL 1
```
cpupower frequency-set -g performance && docker exec -ti  srsran_leonardo_bladerf bash -c 'srsepc'
```
Tape ctrl+shift+T

### ON TERMINAL 2
```
cpupower frequency-set -g performance && docker exec -ti  srsran_leonardo_bladerf bash -c 'srsenb'
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

