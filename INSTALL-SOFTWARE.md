# Software Installation Guide 🛠️

In the previous [INSTALL-HARDWARE-AND-SOFTWARE-PREP](./INSTALL-HARDWARE-AND-SOFTWARE-PREP.md) guide, we already installed the OpenMower Operating System.

This guide describe the remaining software installation steps as well as the mostly required ESC calibration for the drive and mow motors.

<details>
<summary>Activate external WiFi antenna 📶</summary>

If you assembled the adhesive antenna as described in the previous guide, or any other external antenna to the ipex connector of the CM, you also need to activate it.

1. Boot system up
2. Get a terminal via SSH or [WebTerminal](http://openmower:7681)
3. Edit Pi's configuration via: `nano /boot/firmware/config.txt` or `mcedit /boot/firmware/config.txt`
4. Scroll down and change from internal (ant1) to external (ant2) antenna:
   ```
   ...
   [cm4]
   # CM4 antenna selection — choose Wi‑Fi antenna path
   # Internal/on‑module antenna (default):
   # dtparam=ant1
   # External- U.FL/IPEX antenna connector:
   dtparam=ant2
   ```
5. Reboot the system `sudo reboot`

</details>

<details>
<summary>Robot Operating System (open_mower_ros) 🚜</summary>

Easiest way to configure open_mower_ros is by using the awesome openmower-cli.

Get a terminal via SSH or [WebTerminal](http://openmower:7681), do an `openmower configure env` and read carefully all comments, **but** set `VERSION="v2"` for our Sabo build as it contains specific enhancement which aren't in latest as of writing.

Once the editor get saved and closed, openmower-cli will immediately pull the image and start it once fully downloaded.

Lastly, configure your personal settings via `openmower configure ros`. Change at least:
- gps: `datum_lat`, `datum_long` and point it to your planned docking position. You get em by right-click at the location within [Google Maps](https://www.google.com/maps)
- gps: `protocol: "NMEA"` because you most likely use a UM9x GNSS receiver
- Fill in all `ntrip_client` vars

Once the editor get saved and closed, openmower-cli will again immediately restart the openmower service to load the new settings

</details>

<details>
<summary>Calibrate ESCs with your motors 🛠️</summary>

In most cases, the assembled ESC's of a Carrier-Board-Kit have been preconfigured, **but** that configuration is only a rough default configuration, which sometimes sounds ugly and is mostly inefficient as well.

That's why it's highly recommended to do a individual ESC/Motor calibration with your own motors, to get quite as well as efficient running motors!

## Required preparations

1. Unmount the mower blade!
2. Really, unmount the blade! This is a huge mower with a strong motor and large blade!
3. Lift up the back of the mower during calibration with a small carton, so that the wheels are able to spin freely
4. Unmount the mower blade!
5. You need the [VESC Tool Free](https://vesc-project.com/vesc_tool) for your PC. If you don't have it already installed, you need to make an account there, put the "0€ VESC Tool Free" into the cart and checkout. Some seconds later you'll receive a download link.
6. Check if you disassembled the mower blade!

## Reference Informations

### ESC Ports

The ESCs get exposed by our xCore on the following ports:
| xCore Port |               ESC               |
| :--------: | :-----------------------------: |
|   65102    |   Left Drive ESC (wheel icon)   |
|   65103    | Mower ESC (blade or blood icon) |
|   65104    |   Left Drive ESC (wheel icon)   |

### Drive Motor Specs - ECI 42.20 N

| Characteristic | Value    |
| -------------- | -------- |
| Motor Type     | Inrunner |
| Engine Mass    | ~ 330g   |
| Rated Voltage  | 24V DC   |
| Rated Current  | 1.25A    |
| Rated Power    | 30W      |
| Motor Poles    | 6        |

### Mower Motor Specs - ECI 63.20

| Characteristic | Value    |
| -------------- | -------- |
| Motor Type     | Inrunner |
| Engine Mass    | ~ 750g   |
| Rated Voltage  | 24V DC   |
| Rated Current  | 8.5A     |
| Rated Power    | 204W     |
| Motor Poles    | 8        |

### Default ESC configurations

If anything goes wrong or whatever, here are the default [SABO ESCs configs](https://github.com/xtech/hw-openmower-sabo/tree/main/Configs/xESC)


## Drive Motor Calibration

Let's start with left drive motor.

1. Stop open_mower_ros via `openmower stop`
2. Ensure your mowers back is lifted a little bit, so that the wheels can spin freely
3. You should do all calibration on battery power and not with docked power
4. To get connected to xCores exposed ESC port from outside the robot, we need to relay it to our accesible IP of our Pi. We can do this via `socat TCP-LISTEN:65102,fork TCP:172.16.78.150:65102` in a terminal on our Pi (SSH or [WebTerminal](http://openmower:7681)), whereas '172.16.78.150' should be the internal IP of your xCore.
5. Start VESC Tool
6. Connect to mowers ESC: Adapt your hostname or IP accordingly, set the socat'ted ESC port and click connect:<br>
   ![Connect](assets/vesc_1_connect.jpg)<br>
   You'll probably get an "old firmware" warning, confirm it, it's harmless:<br>
   ![Old FW](assets/vesc_2_fw_old.jpg)

7. Enable realtime data: Later on, we wanna validate our calibration with a known reference value, but also during calibration it's interested to see the displayed values in the marked 2 window:<br>
   ![RT Data](assets/vesc_3_realtime_data.jpg)
8. Start the FOC Calibration Wizard:<br>
   ![Start FOC Calibration](assets/vesc_4_voc_1.jpg)<br>

   Now we need to provide some specs of our motor. **These are the specs for the left and right drive motors, for the mow motor, we need to use other specs**:<br>
   <img src="assets/vesc_4_voc_2.jpg" style="vertical-align: middle; width:31%"> 🡆 <img src="assets/vesc_4_voc_3.jpg" style="vertical-align: middle; width:31%"> 🡆 <img src="assets/vesc_4_voc_4.jpg" style="vertical-align: middle; width:31%"><br>

   <img src="assets/vesc_4_voc_5.jpg" style="vertical-align: middle; width:31%"> 🡆 <img src="assets/vesc_4_voc_6.jpg" style="vertical-align: middle; width:31%"> 🡆 <img src="assets/vesc_4_voc_7.jpg" style="vertical-align: middle; width:31%"><br>

   Once calibration has been done, **do not change the direction** (even though the left wheel turns forward during calibration, whereas the right one backwards):

   <img src="assets/vesc_4_voc_8.jpg" style="vertical-align: middle; width:31%">
9. Now that the calibration succeed, lets test the result:
   ![Run Test](assets/vesc_5_test.jpg)<br>

   Test with "**D 0,4**" and press the "Duty cycle" play button. If it draw **<= 0.15A** and sound healty, it is calibrated well.<br>
   Test with some higher duty settings. It will become more loud for sure, but should always spin smooth and sound healty. If not, press the STOP sign.

10. As a last important step, load the correct ESC-App config via: _File → Load App Configuration XML_, choose `SABO_Drive-App.xml` (see [SABO ESCs configs](https://github.com/xtech/hw-openmower-sabo/tree/main/Configs/xESC)) and finally press the `↧A` icon (Write app configuration) on the right side.

11. Optional misc settings which you might align to be within the motor/battery specs:<br>
   ![Drive Settings](assets/vesc_6_settings1.jpg)

Done :satisfied:<br>
... **but not finished** :v: ... you need to do the whole procedure again, but with the right drive side.

So, <kbd>Ctrl</kbd>+<kbd>c</kbd> your socat, do it again but for port 65104 and start over once more at point 4. but with port 65104 to do the same for the right ESC


## Mow Motor Calibration

For the mow motor ESC calibration, you do the same workflow, but with adapted values:

1. socat/VESC connect port is 65103
2. During FOC Calibration Wizard use the following values:
   - Tab "Motor" = Medium Inrunner ~750g
     - Advanced: Max Power Loss = 200, Motor Poles = 8
   - Tab "Battery"
     - Battery Capacity = 3.9Ah
   - Tab "Setup"
     - Gear Ratio = Check Direct Drive
     - Motor Poles = 8
     - Motor Temp. Sensor = disabled

3. Test with "**D 0,08**" which should draw **<= 0.52A** (without assembled blade)
4. Check/Adjust blade rotation direction:<br>
   We need to ensure that the blade is rotating CCW (when watching from downside onto the axis). Do this with a slow rotation speed like "D 0,08".

   If it rotates CW, change direction via: _Motor Settings → General → Tab General → Invert Motor Direction_. **Do not forget to do: "Write motor configuration" via `↧M`**

5. Load the correct ESC-App config via: _File → Load App Configuration XML_, choose `SABO_Mower-App.xml` (see [SABO ESCs configs](https://github.com/xtech/hw-openmower-sabo/tree/main/Configs/xESC)) and finally press the `↧A` icon (Write app configuration) on the right side.

6. Limit blade RPM:<br>
   It's important to limit the max. RPM to the one like OEM is running it! Otherwise you risk your motor bearings or more dangerous: Your blade might fly away (totenkopf symbol)
   ![Limit RPM](assets/vesc_7_mow_settings2.jpg)

7. Optional misc settings which you might align to be within the motor/battery specs:<br>
   ![Drive Settings](assets/vesc_7_mow_settings1.jpg)

</details>

