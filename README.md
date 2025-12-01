## Brief:

### Software

- Linux : ubuntu 22.04.5 LTS (jammy)
    - Not supported by Casper tools for the Matlab - Vivado combination used here. 
    - We will still continue and see how this goes
- Linux : ubuntu 20.04.1 (officially supported by Casper)
- Linux packages : gcc, libpq-dev
- Matlab : 2021a
- Vivado : 2021.1

### RFSoC Casper environment installation log

#### Python 3.8
- Install python3.8 and python3.8-dev, both required for the casper tools
- Install python3-pip, python3.8-venv, python3-wheel

#### Vivado 2021.1
- Using ubuntu 22.04 - not exactly supported by the caster tools install workflow, but risking it anyway
- Installed Vivado 2021.1 using the Xilinx unified installer. The installer is usually maintained in user Downloads/xilinx
- Linux username must be same as that in JPL's DNS e.g. same as that used for SSO login
- Added JPL Vivado license server __2200@cae-lm-xilinx1__
- If we get error related to __libtinfo__ when trying to start GUI, install it as `sudo apt install libtinfo5`

#### Matlab 2021a
- Installation on JPL computer may require the ISO - using the product installer does not work
- Product keys for older versions can be found at [this wiki page](https://wiki.jpl.nasa.gov/display/plmssa/ECAE+Knowledge+Base+-+MATLAB+FAQ+and+User+Self+Guide)
- The license file __license.lic__ can be found [here](https://opencae.jpl.nasa.gov/portal/#/tool-detail/541531592)
- Modified the __startsg.local__ file with the following additions:

    `export MLIB_DEVEL_PATH=/home/pradhan/jpl/git/sandbox/casper/mlib_devel` # seems to help with the casper and xilinx libraries not being added randomly

- This __startup.m__ should be maintained in the development repo and not the cloned casper repos. use symlink to __startsg__ and locally maintained __startsg__.local` to correctly add all necessary paths while ensuring current Matlab math is the development directory.

- Must be connected to JPL net (e.g. via VPN) to start Matlab with `./startsg`

- Model composer (interface between Simulink and HDL models) does not work as expected.

    - Very similar problems are documented, debugged, and fixed [here](https://strath-sdr.github.io/tools/matlab/sysgen/vivado/linux/2021/01/28/sysgen-on-20-04.html) and [here](https://gist.github.com/dcxSt/13f0760ee423082f15e151170b943fa6)
    
    The two parts to this problem are \
    \
    (1) Warnings about *undefined symbols* and problem with __libhogweed.so*__.
    
    This occurs due to A clash between Matlab using Vivado's version of __libgmp.so__ instead of the system version. The general steps are as follows
     
    (i) `>>!ldd /lib/x86_64-linux-gnu/libhogweed.so.6` # Execute at Matlab prompt.Assumes that the missing headers were from this __libhogweed.so.6__ library
    (ii) `cd [problem-location-dir]` # At bash terminal goto to the location pointed to in the output of `!ldd` that looks to be a __Xilinx__ location and not a system location\    
    (iii) `mkdir exclude` # make a folder to be excluded\
    (iv) `mv -r libhogweed.so* exclude` # move all the conflicting library files into __exclude__ folder
    (v) Repeat (i)-(iv) untill the output of `ldd` shows that Matlab is using system libraries
    \
  
    (2) None of the Xilinx blocks are configurable in simulink design with *socket time out*
    
    This occurs because simulink does not initialize correctly due to needing qt4 libraries. But qt4 is no longer supported for Ubuntu 20 or 22. So we have to first add a PPA for qt4, then force the OS to trust updating from here, and install the necessary packages.

    - Add un-supported PPA repository
    `sudo add-apt-repository add-apt-repository ppa:rock-core/qt4`
    
    - (only for Ubuntu 22) open the apt __*.list__ file, likely __/etc/apt/sources.list.d/rock-core-ubuntu-qt4-jammy.list__ and replace line     
    *deb https://ppa.launchpadbuntucontent.net/rock-core/qt4/ubuntu/ jammy main*\
    with\
    *deb https://ppa.launchpadcontent.net/rock-core/qt4/ubuntu/ focal main*

    - update and install qt4 libraries
        ```
        sudo apt update
        sudo apt install libqtcore4 libqtgui4
        ```


#### Casper tools
[Casper tool flow docs](https://casper-toolflow.readthedocs.io/projects/tutorials/en/latest/tutorials/rfsoc/tut_getting_started.html)


#### Tool flow bugs
[x] endbr64 error with simulating simulink design - related to GNU assembler being out of date compared to the GCC compiler\
Cause 	: This happens because Matlab is trying to use the assembler binary from Xilinx folder v/s the system assembler binaries.\
Debug 	: Compare the output of `which as`and `as -v` at the Linux command prompt and matlab (with ! to run system commands) command prompt.\
Fix	: For now the only fix is to move the Xilinx binary into an *exclude* folder and create a symbolic link in the same location to the system assembler binary e.g. `sudo ln -s /usr/bin/as /[path-to-assembler-binary-used-by-matlab]/as`
 



If *permission denied* error occurs check for folder ownership and permissions. Not a good idea to brute force using *sudo*

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

#### Tool usage


#### Hardware Interface
- Downloaded the SD card image from Casper RFSoC bring-up page [insert link here]
- Setup DHCP on host computer server (Ubuntu) to assign IP address to the RFSoC
- login: casper@[ip-address], password: casper
