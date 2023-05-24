# Crazyflie-BigQuad-Prototype
This repository documents the development of a prototype quadcopter expanded from a Crazyflie 2.1 through the BigQuad expansion deck. The Crazyflie firmware used for development is [version 2023.02](https://github.com/bitcraze/crazyflie-firmware/releases/tag/2023.02).

## Table of Contents
* [Physical Construction](#physical-construction)
	* [Drone Parts](#drone-parts)
	* [Connection Diagram](#Connection-Diagram)

* [Customize Firmware](#Customize-Firmware)
	* [Set up BVM Environment](#Set-up-BVM-Environment)
	* [Installation](#Installation)
	* [USB Permissions](#USB-Permissions)
	* [Radio URI](#Radio-URI)
	* [Firmware Modification](#Firmware-Modification)
		* [BigQuad Driver](#BigQuad-Driver)
		* [Kbuild & Config](#Kbuild-&-Config)
	* [Flashing Firmware](#Flashing-Firmware)
* [ESC Configuration](#ESC-Configuration)
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
Since the BigQuad deck is an early access product, its driver is not enabled in the off-the-shelf firmware. This means that if we do not make custom changes to the firmware, the BigQuad deck will not be detected by Crazyflie when we mount it to the vanilla Crazyflie. Further, the `bigquad.c` driver file in the off-the-shelf firmware also needs to be modified as there is a weird function in `bigquad.c`, namely, `extRxInit()`, that overrides the `setpoint` command generated from the Python API with an external setpoint reader (this bug is discovered by *thefred* in [this thread](https://forum.bitcraze.io/viewtopic.php?t=3931)).

To fulfill the above tasks, we need to use the `kbuild` tool to build and flash the firmware. As explained in [this thread](https://github.com/orgs/bitcraze/discussions/269), the relevant instructions given in the [Bolt and BQ deck - Workshop (Youtube Video)](https://youtu.be/xiWLhr-HpG8) and in the official [BigQuad Product Page](https://www.bitcraze.io/products/bigquad-deck/) are already outdated and creating a `config.mk` file will not work. According to [this blog](https://www.bitcraze.io/2022/02/a-new-way-to-configure-the-crazyflie-firmware/), all firmware developments after the `2022-02 release` should be performed using the `kbuild` build tool, which seems to be only available in a Linux environment. A convenient way to use the `kbuild` tool is to install the `Bitcraze Virtual Machine` (BVM) environment. 

### Set up BVM Environment

#### Installation
1. I followed [this documentation](https://github.com/bitcraze/bitcraze-vm) to install the `Bitcraze Virtual Machine` (BVM). The instructions given in the cited documentation is very detailed and I did not encounter any issues. The only things to note are
   * Make sure to also install the expansion package as it is required for USB connection.
   * If after launching the BVM, only a black screen is shown, just try dragging the window as it forces the desktop to update. The normal desktop should then come out.
2. Click on the `Update all projects` script on the desktop in BVM to update everything. It is very likely that you would get this error,

	```
	Preparing wheel metadata ... error
	```
	
	or something similar. As seen in [this discussion thread](https://github.com/orgs/bitcraze/discussions/545), the above error is caused by the fact that VirtualBox no longer comes with the latest release of `pip3`. Use the following commands to update the dependencies (install one at each time and run the `Update all projects` script to see if the issue is resolved):

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


#### USB Permissions
To connect to the Crazyflie, we need to know its radio URI. The easiest way to determine the radio URI now (see [this thread](https://github.com/orgs/bitcraze/discussions/660) for detail) is connecting the drone to the PC with a micro USB cable and then use the `scan` button in the Python client. To be able to do so, we need to modify the USB permission rules in the `BVM` with the following steps:

1. In BVM, open the `Terminal Emulator` app;
2. Type the following commands in the terminal:

	```
	sudo groupadd plugdev
	sudo usermod -a -G plugdev $bitcraze
	```
3. Reboot BVM;
4. Click on the `Home` icon on the desktop;

	<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/update_udev_rules_1.png" width="200">

5. Navigate to this path: `~\etc\udev\rules.d\`. Then, right-click on the `99-crazyradios.rules` file and in the drop down menu select `Open Terminal Here`. In the new terminal window, type the following command to delete the `99-crazyradios.rules` file:
	```
	sudo rm -rf 99-crazyradios.rules
	```
	Next, open the files `99-crazyflie.rules` by double-clicking on its icon. 

	<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/update_udev_rules_2.png" width="500">

6. In the `.rules` file, clear any existing content and then paste the following lines:

	```
	cat <<EOF | sudo tee /etc/udev/rules.d/99-bitcraze.rules > /dev/null
	# Crazyradio (normal operation)
	SUBSYSTEM=="usb", ATTRS{idVendor}=="1915", ATTRS{idProduct}=="7777", MODE="0664", GROUP="plugdev"
	# Bootloader
	SUBSYSTEM=="usb", ATTRS{idVendor}=="1915", ATTRS{idProduct}=="0101", MODE="0664", GROUP="plugdev"
	# Crazyflie (over USB)
	SUBSYSTEM=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0664", GROUP="plugdev"
	EOF
	```

    Save the file. 

7. Reload the `udev-rules` by typing the following commands in the terminal:

	```
	sudo udevadm control --reload-rules
	sudo udevadm trigger
	```

8. Reboot the BVM. Done.


#### Radio URI
We do not need to know the radio URI of the Crazyflie to flash the firmware. However, for the completeness of this documentation, the steps for determining the radio URI are noted below.
1. Connect the Crazyflie to the PC using a micro USB cable.

	<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/usb_connected_drone.png" width="200">

2. Open the `cfclient`. Then look to the bottom right corner of the BVM, click on the highlighted icon, and choose `Bitcraze AB Crazyflie 2.X [0200]`. This allows the BVM to see the cabled drone.

	<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/usb_vm.png" width="550">

3. In the `cfclient`, click on the `Scan` button. At this stage, the `Address` does not need to be the actual URI. After scanning, the `Select an interface` should display `usb://0`, meaning that the Python client has found the cabled drone. Now click on the `Connect` button.

	After connected, go to the `Connect` option in the top panel, then in the drop down menu, choose `Configure 2.X`.
	
    In the popped-up window, the current `Radio Address` of the cabled drone is displayed. We can also overwrite that address by changing that field, and then click on the `Write` button. 
	
	<img src="https://github.com/Gloogger/Crazyflie-BigQuad-Prototype/blob/main/images/radio_uri.png" width="700">



### Firmware Modification

#### BigQuad Driver
To send control setpoints with the commander framework (e.g. `send_setpoint()`) in the Python API, we need to comment out the default setpoint reader function, {\tt extRxInit()}, in the {\tt BigQuad} driver. `extRxInit()` is a function that loads an external setpoint reader with a higher priority that overrides whatever control setpoints the Python AIP has assigned (see [this thread](https://forum.bitcraze.io/viewtopic.php?t=3931) for detail). 

To comment out `extRxInit()`, follow the steps below:
1. In the BVM, navigate to the `bigquad.c` file located at `Desktop/projects/crazyflie-firmware/src/deck/drivers/src/bigquad.c`.

3. Comment out line 85 (as in the 2023-02 release) which has the `extRxInit()` code. Because the `bigquad.c` file receives  update from time to time, lines 83-88 are provided below to give context.
	```
	DEBUG_PRINT("Switching to brushless.\n");
	motorsInit(motorMapBigQuadDeck);
	extRxInit(); // <- comment out this line

	// Ignore charging/charged state to allow low-battery warning.
	pmIgnoreChargedState(true);
	```

4. Save the file. Done!



#### Kbuild & Config

### Flashing Firmware

## Test Flight


