## Brief:

### RFSoC Casper environment installation log

#### Python 3.8
- Install python3.8 and python3.8-dev, both required for the casper tools

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
- Modified the `startsg.local` file with the following additions:\
`export LD_PRELOAD=${LD_PRELOAD}:"/lib/x86_64-linux-gnu/libgnutls.so.30";matlab` #likely that this does not do anything\
`export MLIB_DEVEL_PATH=/home/pradhan/jpl/git/sandbox/casper/mlib_devel`#seems to help with the casper and xilinx libraries not being added randomly
