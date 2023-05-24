# Crazyflie-BigQuad-Prototype
This repository documents the development of a prototype quadcopter expanded from a Crazyflie 2.1 through the BigQuad expansion deck. The Crazyflie firmware used for development is [version 2023.02](https://github.com/bitcraze/crazyflie-firmware/releases/tag/2023.02).

## Table of Contents
* [Physical Construction](#physical-construction)
  * [Drone Parts](#drone-parts)
  * [Connection Diagram](#Connection-Diagram)
  * [ESC Configuration](#ESC-Configuration)
* [Customize Firmware](#Customize-Firmware)
  * [Set up BVM Environment](#Set-up-BVM-Environment)
  * [Installation](#Installation)
  * [USB Permissions](#USB-Permissions)
  * [Radio URI](#Radio-URI)
  * [Firmware Modification](#Firmware-Modification)
    * [BigQuad Driver](#BigQuad-Driver)
    * [Kbuild & Config](#Kbuild-&-Config)
  * [Flashing Firmware](#Flashing-Firmware)
* [Test Flight](#Test-Flight)




## Physical Construction
This section documents the hardware assembly of the prototype quadcopter as well as the relevant configurations.

### Drone Parts
Items used to assemble the prototype drone is given in the table below.
| Category        | Model                                                       | 
| :-------------: | :----------------------------------------------------------:| 
| Frame           | DJI F450 4-Axis AirFrame                                    | 
| Motors          | DJI Phantom 1/2/3 2312 Brushless Motors                     |  
| ESC             | Hobbywing ESC X-Rotor Series H-King 20A 3-4S Lipo (No BEC)  |  
| Flight Control  | Crazyflie 2.1 + BigQuad deck                                |
| Battery         | HobbyLine 4S1P 100C 1500mAh Lipo Battery                    |
| Propellers      | DJI Phantom Series 9450 Self-Locking Propellers             |

### Connection Diagram
Following the basic connection diagram given in the [BigQuad Product Page](https://www.bitcraze.io/products/bigquad-deck/), the ESCs (Electronic Speed Controllers), the BigQuad deck, and the motors are connected as shown below.

<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/raw/main/images/basic_connection.png" width="550">

The assembled prototype is shown in the Figure below.

<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/assembled_drone.jpg" width="550">



## Customize Firmware
Since the BigQuad deck is an early access product, we need to modify the Crazyflie firmware so as to enable some features. Firstly, we need to enable the BigQuad deck driver that is not enabled by default, and secondly, we need to modify the `bigquad.c` driver file in order to use the Python API.

To complete the above tasks, we need to use the `kbuild` tool to build and flash the firmware. According to [this GitHub discussion thread](https://github.com/orgs/bitcraze/discussions/269) and [this blog](https://www.bitcraze.io/2022/02/a-new-way-to-configure-the-crazyflie-firmware/), all firmware developments after the $2022-02$ release should be performed using the `kbuild` build tool, which seems to be only available in a Linux environment. A convenient way to use the `kbuild` tool is to install the `Bitcraze Virtual Machine` (BVM) environment. 


### Set up BVM Environment
1. I followed [this documentation](https://github.com/bitcraze/bitcraze-vm) to install the `Bitcraze Virtual Machine` (BVM). The instructions given in the cited documentation is very detailed and I did not encounter any issues. The only things to note are
   * Make sure to also install the expansion package as it is required for USB connection.
   * If after launching the BVM, only a black screen is shown, just try dragging the window as it forces the desktop to update. The normal desktop should then come out.
2. Click on the `Update all projects` script on the desktop in BVM to update everything. It is very likely that you would get this error,
```
Preparing wheel metadata ... error
```
   This error is caused by the fact that VirtualBox no longer comes with the latest release of `pip3`. Use the following commands to update the dependencies (install one at each time and run the `Update all projects` script to see if the issue is resolved):
```
pip3 install --upgrade pip
sudo apt update
pip3 install PyQt5 // only 0.23.0 version or compatible version works with cfclient
pip3 install qasync~=0.23.0
sudo apt-get install packer
```
   The installation may require user name and password:
   * User: bitcraze
   * Pass: crazyflie
 
    
3. After successful update, we should be able to call out the Python client either by typing `cfclient` in the `Terminal Emulator` app whose icon is on the desktop, or by clicking on the `Crazyflie client` icon which is on the desktop.

<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/cfclient_in_BVM.png" width="450">



#### Installation
blah blah

#### USB Permissions

#### Radio URI

### Firmware Modification

#### BigQuad Driver

#### Kbuild & Config

### Flashing Firmware

## Test Flight


