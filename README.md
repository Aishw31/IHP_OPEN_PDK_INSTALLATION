Setup of IHP 130nm BiCMOS Open Source PDK
==========================================
Recommended Environment:
------------------------
- Ubuntu Linux 20

# Requirements

Run the requirement.sh file 
```bash
    ./requirements.sh
```


## autoconf >= 2.70 is required for Ngspice
```bash
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.71.tar.gz
tar -xvzf autoconf-2.71.tar.gz
cd autoconf-2.71
./configure
make
sudo make install
autoconf --version
cd ..
rm -rf autoconf-2.71
```


# Steps
## 1. Clone the PDK Repository:
----------------------------
```bash
mkdir ~/ihp
cd ~/ihp 
git clone --recursive https://github.com/IHP-GmbH/IHP-Open-PDK.git
cd IHP-Open-PDK
git checkout dev
```
NOTE:
- Use the `dev` branch because the ngspice example may not work in `main`.
- The `--recursive` flag ensures submodules are included.

## 2. Set Up Environment Variables:
--------------------------------
Add the following lines to your ~/.bashrc file:

```bash
gedit ~/.bashrc
# or use 
# nano or vim
# And then append the following lines
```
```bash
export PDK_ROOT=$HOME/ihp/IHP-Open-PDK
export PDK=ihp-sg13g2
export KLAYOUT_PATH="$HOME/.klayout:$PDK_ROOT/$PDK/libs.tech/klayout"
export KLAYOUT_HOME=$HOME/.klayout
```
Then run:
```bash
source ~/.bashrc
```
## 3. Install NGSPICE:
-------------------
```bash
cd ~/ihp
git clone https://git.code.sf.net/p/ngspice/ngspice ngspice-ngspice
cd ngspice-ngspice
./autogen.sh
./configure --enable-osdi
make
sudo rm /usr/bin/ngspice
sudo ln -s ~/ihp/ngspice-ngspice/src/ngspice /usr/bin/ngspice
cd ~
```


## Generate libraries related to osdi and append .spiceinit 

Before that You need openvaf installed
```bash
cd ~/ihp
wget https://openva.fra1.cdn.digitaloceanspaces.com/openvaf_23_5_0_linux_amd64.tar.gz
tar -xvzf openvaf_23_5_0_linux_amd64.tar.gz
rm -rf openvaf_23_5_0_linux_amd64.tar.gz
sudo ln -s ~/ihp/openvaf /usr/bin/openvaf
cd ~
```

Now lets Do the rest

```bash
cd $PDK_ROOT/$PDK/libs.tech/verilog-a
./openvaf-compile-va.sh 
cd ../ngspice
cat .spiceinit >> ~/.spiceinit
cd ~
```

#  Run Basic Simulation:
------------------------
## Example 1: NPN HBT Simulation
-----------------------------
Save the following netlist as `dc_hbt_13g2.spice`:
```ngspice
**.subckt dc_hbt_13g2
Vce net3 GND 1.2
I0 GND net1 1u
Vc net3 net2 0
.save i(vc)
XQ1 net2 net1 GND GND npn13G2l Nx=1 le=1.0e-6

.lib cornerHBT.lib hbt_typ

.param temp=27
.control
save all
op
print I(Vc)
.endc

.GLOBAL GND
.end

```

Run with:
```bash
ngspice -b dc_hbt_13g2.spice
```
Expected Output:
```
Note: Compatibility modes selected: hs a


Circuit:

Doing analysis at TEMP = 27.000000 and TNOM = 27.000000

Using SPARSE 1.3 as Direct Linear Solver

No. of Data Rows : 1
i(vc) = 6.492800e-04
Note: Simulation executed from .control section
```

## Example 2: NMOS Simulation
--------------------------
Save the following netlist as `mostest.spice`:
```ngspice
.lib cornerMOSlv.lib mos_tt
Vgs net1 GND 0.4
Vds net3 GND 1.0
Vd net3 net2 0
.param temp=27
XM1 net2 net1 GND GND sg13_lv_nmos w=1.0u l=0.13u ng=1 m=1
.control
save all
op
let Id = @n.xm1.nsg13_lv_nmos[ids]
print Id
.endc
.GLOBAL GND
.end
```
Run with:
```bash
ngspice -b mostest.spice
```
### Expected Output:
```
Note: Compatibility modes selected: hs a

Warning: m=xx on .subckt line will override multiplier m hierarchy!


Circuit: mostest

Doing analysis at TEMP = 27.000000 and TNOM = 27.000000

Using SPARSE 1.3 as Direct Linear Solver

No. of Data Rows : 1
id = 1.145621e-06
Note: Simulation executed from .control section```

