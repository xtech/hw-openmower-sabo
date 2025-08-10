# Hardware Installation and Software Preparation Guide

This guide describes the hardware installation and the software preparation steps required during installation for the OpenMower‑V2 Carrierboard for SABO mowers.

Scope

- Primary focus: mechanical/electrical installation of the Carrierboard and modules (xCore + CM4, UM9x, antennas, connectors) inside the mower.
- Included: software preparation that must be performed during hardware install (flashing OpenMowerOS onto CM4, first boot, basic connectivity checks).
- Not included: the full software setup, ROS configuration, and calibration. For those steps, see INSTALL-SOFTWARE.md.

At a glance (process overview)

1) Open the mower and remove the OEM mainboard.
2) Prepare xCore & CM4 (flash OpenMowerOS; first boot; verify Wi‑Fi/SSH/WebApp).
3) Prepare the UM9x RTK module.
4) Install the Wi‑Fi adhesive antenna.
5) Assemble the Carrierboard and the mainboard holder; connect plugs.
6) Install the HA/HX‑901 GPS antenna on the cover; seal outside and inside.
7) Close the housing and finish assembly.

## 1. Open the mower and remove the OEM mainboard

- Place the mower on its back and remove the six 10 mm screws from the housing.
- Remove the blade with the FIXME: 6?mm Allen key (right‑hand thread). Use gloves and secure the blade while loosening.
- Hold the housing together with both hands and place the mower back on its wheels.
- Slightly open the cover on the handle (rear) side by a few centimeters.
- Using a flashlight, disconnect the display ribbon cable(s):
  - Series‑I: two ribbon cables going to the CoverUI. Each plug has small side locks; press to release, then unplug from the mainboard.
  - Series‑II: one ribbon cable; pull it straight out from the mainboard (no release locks).
- You can now open the cover further. A single cable from the cover’s charging contacts goes toward the mainboard. Before the mainboard there is a 2‑pin Molex plug; press to unlock and disconnect it. Put the cover aside for later modification and reassembly.
- Disconnect all remaining cables from the OEM mainboard. Many plugs have a latch; press to release before pulling.
- The OEM mainboard is mounted on a black plastic mainboard holder. On its back side are two larger Torx T30 screws. Remove them to lift out the mainboard together with the holder.
- Remove the OEM mainboard from the holder by unscrewing the Torx T20 screws.


## 2. Prepare xCore & CM4

- Optional: Waveshare CM4‑Heatsink‑B (SKU 22097)
  - Place the small standoffs on the xCore’s CM4 mounting holes.
- Carefully seat the Raspberry Pi CM4 onto the xCore connector. Ensure the orientation is correct (the small IPEX antenna connector aligns appropriately with the SODIMM side).

- Download OpenMowerOS from [OpenMowerOS Releases](https://github.com/ClemensElflein/OpenMowerOS/releases) and expand the Assets section.
- If not already installed, install the [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

- Option A: Raspberry Pi CM4 Lite (without eMMC storage)
  - Insert a 32 GB or larger microSD card into your PC.
  - Start Raspberry Pi Imager.
  - Choose Model = “Raspberry Pi 4”.
  - Choose OS = “Use custom” and select the downloaded OpenMowerOS image.
  - Choose your inserted microSD card. **Double‑check you selected the correct removable device, not a hard drive. All data on the selected device will be erased.**
  - Click Next. Do not enable the optional custom settings. Write the image.
  - When finished, remove the microSD card and insert it into the xCore.

- Insert your xCore+CM4 assembly into the SABO Carrierboard and temporarily mount it to the black plastic mainboard holder (from section 1) using only 2–4 screws.
- Put the assembly back into the mower, but do not fasten it yet.
- Plug in the large power Molex cable. Make sure to use the correct Series‑I / Series‑II connector matching your mower model.

- Option B: Raspberry Pi CM4 with eMMC storage
  - Connect a Micro‑USB cable from your PC to the xCore’s Micro‑USB port.
  - Install the “rpiboot” host utility as described in [Set up the host device](https://www.raspberrypi.com/documentation/computers/compute-module.html).
  - Power on[^1] the mower while pressing the xCore “Rpi Boot” button. When the status LEDs start flashing, release the button.
  - Run `sudo rpiboot` on your PC (see the Raspberry Pi docs for Windows/macOS). After a few seconds a new mass‑storage device (the CM4 eMMC) appears.
  - Start Raspberry Pi Imager.
  - Choose Model = “Raspberry Pi 4”.
  - Choose OS = “Use custom” and select the downloaded OpenMowerOS image.
  - Choose the eMMC device just mounted by rpiboot. **Double‑check you selected the correct device. All data will be erased.**
  - Click Next. Do not enable the optional custom settings. Write the image.
  - When verified, close the imager, unplug USB, and power the mower off again.

- First boot
  - Power on[^1] the mower.
  - The CM4 will reboot several times; this may take a few minutes. It will create an “OpenMower‑<nnn>” Wi‑Fi access point. Follow OpenMower’s guide: [Configuring Wi‑Fi](https://openmower.de/docs/software-setup/configuring-wifi/).
  - After Wi‑Fi configuration, the CM4 will connect to your local network and download the latest open_mower_ros image. This can take 20–40 minutes.
  - You may already check connectivity: `ping openmower.local` and `ssh openmower@openmower.local` (password: `openmower`).
  - When the open_mower_ros image is installed (after about 30–50 minutes), open the OpenMower WebApp at `http://openmower.local:8080` (note http, not https).
  - Proceed only when ping, SSH, and the WebApp all work.

- Power the mower off, unplug the large power Molex from the Carrierboard, and remove the assembly again.

- Optional: Waveshare CM4‑Heatsink‑B (SKU 22097)
  - Remove the xCore from the Carrierboard.
  - Remove the Carrierboard from the black plastic mainboard holder.
  - Place the thermal pads on the CM4 chips as shown:<br>
    ![WaveShare Thermal Pads](assets/CM4-HEATSINK-B-ThermalPads.jpg)
  - Place the heatsink on top and fasten only the two screws on the SODIMM side.
  - Place the larger Waveshare standoff on the Carrierboard where the xCore mounts.
  - Plug the xCore into the Carrierboard and use the longer screws to fasten xCore + Carrierboard as shown:<br>
    ![WaveShare Assembled Spacer](assets/CM4-HEATSINK-B-AssembledSpacer.jpg)

- Do not fully assemble the Carrierboard + mainboard holder yet.

  
## 3. Prepare the UM9x RTK module

- Connect the UM9x to your PC using the supplied USB cable.
- Open a serial terminal (minicom, miniterm, CuteCom, etc.) at 115200 baud.
- Send the following commands:
  ```
  FRESET
  CONFIG COM1 921600
  ```
  This resets the UM9x to factory defaults and sets COM1 to 921600 baud.
- Reconnect your terminal at 921600 baud.
- Then send:
  ```
  MODE ROVER UAV
  GPGSV COM1 2
  GPRMC COM1 1
  GPGSA COM1 1
  GPVTG COM1 1
  GPGST COM1 1
  GPGGA COM1 0.2
  SAVECONFIG
  ```
- Disconnect and mount the UM9x module onto the Carrierboard. You may need to solder straight pin headers first.
- Install the IPEX/SMA cable (usually included with the UM9x) as shown:<br>
  ![UM9x IPEX/SMA cable](assets/UM9x-IPEX-SMA-Cable.jpg)


## 4. Install Wi‑Fi adhesive antenna

- I normally place it like this:<br>
    ![WLAN adhesive antenna](assets/om-sabo-cb-wlan-antenna1.jpg)<br>
    ![WLAN adhesive antenna](assets/om-sabo-cb-wlan-antenna2.jpg)

## 5. Assemble Carrierboard and mainboard holder

- Now that all modules are prepared and mounted, fasten the Carrierboard to the mainboard holder. If you have fewer screws than holes, prioritize holes close to the connectors.
- Mount the Carrierboard + mainboard holder back into the mower and fasten it with the two screws.
- Carefully connect all plugs. Some plugs fit into multiple counterparts, verify the labels or use this plug overview:<br>
  
  |                     Series-I Plugs                     |
  | :----------------------------------------------------: |
  | ![Series-I Plugs](assets/om-sabo-cb-s1-v02-plugs.jpg)  |
  |                  **Series-II Plugs**                   |
  | ![Series-II Plugs](assets/om-sabo-cb-s2-v02-plugs.jpg) |

  Note: some plugs are rotated—do not force any connector.

## 6. Install the HA/HX‑901 GPS antenna on the cover

- Drill a 5 mm hole in the cover approximately at the position shown here:<br>
  ![GPS antenna 1](assets/om-sabo-gps-antenna1.jpg)
- Install the SMA extension cable and seal it from the top with silicone or a similar sealant. Don’t use too much, so the HA/HX‑901 can still be screwed on later.
- Also seal the inside thoroughly to prevent any water ingress. On the inside, more is better:<br>
  ![GPS antenna 2](assets/om-sabo-gps-antenna2.jpg)
- Allow sufficient time for the sealant to cure before proceeding.

## 7. Close the housing

- Place the cover back onto the mower. Let the front engage slightly, but keep the rear open enough to:
  - Connect the GPS antenna cable to the Carrierboard.
  - Reconnect the docking‑contact cable.
  - Finally, reconnect the CoverUI ribbon cable(s).
- Close the cover completely so it fits and latches evenly all around.
- Hold the housing together with both hands and turn the mower back onto its back.
- Reinstall the six 10 mm hex‑head screws.
- Do not install the blade yet.
- Put the mower back on its wheels.
- Finally, screw the HA/HX‑901 antenna onto the cover.


---

TODO: For software installation, see [INSTALL-SOFTWARE.md](INSTALL-SOFTWARE.md).

[^1]: If your battery is drained (common for second‑hand mowers), you can power the Carrierboard with 12–36 V DC via the 2‑pin Molex you unplugged when removing the cover. Red is +, black is −. **Mind the current draw.** If your DC PSU cannot supply about 2.65 A, disconnect the Molex plug to the battery to prevent charging current from overloading the PSU.
