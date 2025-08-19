## Brief:

### RFSoC Casper environment installation log

#### Python 3.8
- Install python3.8 and python3.8-dev, both required for the casper tools
- Install python3-pip

#### Vivado 2021.1
- Using ubuntu 22.04 - not exactly supported by the caster tools install workflow, but risking it anyway
- Installed Vivado 2021.1 using the Xilinx unified installer
- Linux username must be same as that in JPL's DNS e.g. same as that used for SSO login
- Added JPL Vivado license server `2200@cae-lm-xilinx1`
- If we get error related to `libtinfo` when trying to start GUI, install `libtinfo5`

#### Matlab 2021a
- Installation on JPL computer may require the ISO - using the product installer does not work
- Product keys for older versions can be found at [this wiki page](https://wiki.jpl.nasa.gov/display/plmssa/ECAE+Knowledge+Base+-+MATLAB+FAQ+and+User+Self+Guide)
- The license file `license.lic` can be found [here](https://opencae.jpl.nasa.gov/portal/#/tool-detail/541531592)
- Modified the `startsg.local` file with the following additions:
```
export LD_PRELOAD=${LD_PRELOAD}:"/lib/x86_64-linux-gnu/libgnutls.so.30";matlab #likely helping with finding correct libraries
export MLIB_DEVEL_PATH=/home/pradhan/jpl/git/sandbox/casper/mlib_devel#seems to help with the casper and xilinx libraries not being added randomly
```
- Modified local `startup.m` from repo with following path addition `addpath` for the model composer simulink libraries.

`addpath([getenv('COMPOSER_PATH'), '/simulink']); # Needed to correctly load the Xilinx toolbox as a simulink library`

- This `startup.m` should be maintained in the development repo and not the cloned casper repos. use symlink to `startsg` and locally maintained `startsg.local` to correctly add all necessary paths while ensuring current Matlab math is the development directory.

- Must be connected to JPL net (e.g. via VPN) to start Matlab with `./startsg`

#### DHCP setup
- The `casper` SD image configures onboard Linux OS to use DHCP, this means that IP address will be provided by the host computer
- A DHCP server should be set up on the host computer [Link](https://medium.com/@sydasif78/setting-up-a-dhcp-server-on-ubuntu-a-guide-for-network-engineer-d620c5d7afb2)

- Preliminaries:
```
sudo apt install netplan.io isc-dhcp-server # Install packages. The netplan.io package may already be installed

sudo ip addr show # Get the name of desired interface e.g. enp0s31f6

nano /etc/netplan/[config-filename].yaml # Configure the Netplan YAML
```
Host computer Netplan for desired interface [Example](https://documentation.ubuntu.com/server/explanation/networking/configuring-networks/):
```
network:
    version: 2
    renderer: NetworkManager
    ethernets:
        enp0s31f6
            dhcp4: no
            addresses: [192.168.137.1/24]
            nameservers:
                addresses: [192.168.137.1,8.8.8.8]
            routes:
                - to: default
                  via: 192.168.137.1
```
`sudo netplan apply` # apply configuration

- DHCP configuration on host computer
```
sudo systemctl status isc-dhcp-server #check DHCP service status

sudo nano /etc/dhcp/dhcpd.conf #Edit the DHCP config. file
```
- Host computer DHCP configuration [Example](https://medium.com/@sydasif78/setting-up-a-dhcp-server-on-ubuntu-a-guide-for-network-engineer-d620c5d7afb2):
```
default-lease-time 43200;
max-lease-time 86400;
option subnet-mask 255.255.255.0;
option broadcast-address 192.168.137.255;
option domain-name "local.lan";
authoritative;
subnet 192.168.137.0 netmask 255.255.255.0 {
    range 192.168.137.100 192.168.137.200;
    option routers 192.168.137.1;
    option domain-name-servers 192.168.137.1;
}
```
`nano /etc/default/isc-dhcp-server` # edit which interface the DHCP server should be service\
`INTERFACES="enp0s31f6"` # assuming the interface name is __enp0s31f6__\
`sudo systemctl start isc-dhcp-server` # start server
`cat /var/lib/dhcp/dhcpd.leases` # check which IP addresses have been leased out (when clients are connected to server over ethernet interface)


#### Hardware Interface
- Downloaded the SD card image from Casper RFSoC bring-up page [insert link here]
- Setup DHCP on host computer server (Ubuntu) to assign IP address to the RFSoC
- login: casper@[ip-address], password: casper
