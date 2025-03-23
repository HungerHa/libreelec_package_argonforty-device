# Argon ONE Control add-on (formerly known as: ArgonForty Device Configuration)

Installs services to manage ArgonForty devices such as power button, fan speed and Argon REMOTE.

This will also enable I2C, IR receiver and UART.

**Important:** Starting with version 1.1.6 the add-on has been renamed to "Argon ONE Control". The add-on ID has been changed as well. This was a prerequisite for the integration of this add-on into the LibreELEC add-on repo. For better user experience if beginning from scratch, its now possible to install it from there directly. To distinguish my fork of the add-on from the official, currently outdated version 0.0.1 of Argon40, I have also changed the name of the add-on. Unfortunately, as a one-off event, this change requires the removal of an existing version in advance so as not to collide with running background processes.

## What it does

- supports LibreELEC 11 / 12 / 13
- supports Argon ONE V1/2 (RPi4)
- supports Argon ONE V3 (RPi5)
- probably also supports Argon Fan HAT, but untested (feedback is welcome)
- enables IR receiver (V2/V3, or if self added to V1 pcb)
- enables Argon REMOTE support (rc_maps + keymap)
- fan control with fan curves CPU, SSD/NVMe, GPU and PMIC
- graceful shutdown (power button commands: Reboot , Shutdown ...)

For full support of the power button commands with a RPi5, please use LE12.

It might also work with LE10 (RPi4) but is untested. If someone is using LE10 and it doesn't work, they can try version v.0.0.4: [LibreELEC Thread](https://forum.libreelec.tv/thread/27360-rpi4b-argon-one-case-shutdown/?postID=182477#post182477)

### Known issues

There is a limitation in the Argon ONE case firmware.

After the power button at remote control or the button (held for > 3 seconds, but < 5 seconds) on the back of the case was pressed, KODI including all OS processes only has ~10 seconds to shutdown properly! Once initiated, the 10 seconds power cut timeout can't be interrupted and is perhaps only with another case firmware correctable.

To compensate that, I have optimized v0.0.10+ as far I currently could to decrease the shutdown time to below 8 seconds. Since LibreELEC 12 it taked much more time to shutdown the KODI process. The cause of the delay seems the switch to lgpio / gpiozero as the default python modules to interact with the GPIO chip. A KODI add-on thread with lgpio / gpiozero imported can't be terminated within 5 seconds. KODI tries to kill this thread and afterwards stucks until a timeout of 30 seconds.

Starting with v1.1.4 the add-on supports gpiod as an alternative way to interact with the GPIO pins. This may it possible to stop KODI within 5-6 seconds and properly shutdown via power button of the remote control again!

During the review phase of the integration of this library into the official LibreELEC add-ons, it was decided to integrate the Python bindings (gpiod) into the system-tools add-on instead of the rpi-tools add-on to make it available for all platforms, not just the RPi. As of v1.1.5, both variants are supported.

NOTE: The change will only take effect if KODI has already been started with the updated rpi-tools/system-tools. Restart LibreELEC after updating the rpi-tools/system-tools.

The PRs for system tools have been applied and manual installation is no longer required. Only for historical reason you can find an update version of rpi-tools in the release area of version 1.1.4. which already contains gpiod.

- virtual.rpi-tools-12.0.0.1.zip for LE12
- virtual.rpi-tools-12.80.1.1.zip for LE13 nightly builds

**Note**: Depending on what is currently installed and running in KODI, it sometimes takes longer than 10 seconds (including the final processes of the underlying operating system), then the shutdown will not be graceful and in the worst case data corruption is possible.

*The safety way*: If the power menu of KODI is used to shutdown, the power cut is initiated just at the end (+10 seconds).

### Workaround after improperly shutdown

Situation: Remote control power button pressed → 10 seconds timer starts -> shutdown is initiated → red LED turns off, but it seems that it doesn't respond to the power button on the remote to get it on again.

Please try in this situation (unsuccessful shutdown):

- The red LED must be off
- Press the power button on the remote control
- Wait 10 seconds (MCU cuts off power internally)
- Press the power button on the remote control again to boot

## 3rd party remote control

I have switched from lircd to rc_maps/keymaps (thanks to adam.h. for providing the files) to use the modern way with ir-keytable. If you use a custom lircd.conf for your remote control, please make a backup of your remote control configuration and/or place a lock file **before** installation of the add-on to prevent overwriting.

If there is already a rc_maps.cfg file in the ```/storage/.config``` directory, the add-on checks if the needed line to include the argon40.toml file exists. If necessary, an attempt is made to append the needed line.

In the worst case scenario, to prevent the add-on from changing the IR configuration, all you need to do is place an empty lock file in the ```/storage/.config``` directory.

```bash
touch /storage/.config/argon40_rc.lock
```

## Quick installation (not implemented yet)

Search for "Argon ONE control" within the LibreELEC add-on area and install the add-on. All requirements will be downloaded and installed automatically. If you install this add-on the first time, a reboot is required afterwards to activate all interfaces like I2C, UART and IR. Due to the versioning of the automatically created LibreELEC add-ons, a package revision number starting with 0 is also added - for example 1.1.6.**0**. The first 3 digits correspond to the version of the add-on.

**ATTENTION:** If you have already installed a version older than 1.1.6, it is necessary to remove the older version first. If you have customised the settings to your personal requirements, these will unfortunately be lost in this way. This is a one time event and neccessary because of the required change of the add-on id. The simultaneous installation of both add-on versions will otherwise lead to malfunctions or non-functioning add-ons!

## Manual installation

In the following installation instructions, the term "ZIP file" is used for simplicity.
Because there seems to be room for misinterpretation and to avoid future confusion: It doesn't mean the code download button from GitHub, to download the whole source code repository content!

Please use one of the ready to install add-on archives with the name pattern libreelec_argondevice_x.x.x.zip from the [releases](https://github.com/HungerHa/libreelec_addon_argononecontrol/releases).

The installation process will try to add 3 configuration lines to the config.txt to enable the needed modules for I2C, IR and UART. This part is not bullet proofed, because it looks only for the first line. It skips the needed modification if the line "dtparam=i2c=on" is already there. Therefore it could be better to make a backup of ```/flash/config.txt``` before to see the different.

A few things to do. The first 3 steps in square brackets are optional and are only needed if the dependencies cannot be resolved automatically and can usually be skipped:

- *[ install RPi Tools (LibreELEC Repo -> Program Add-ons) ]*
- *[ install System Tools (LibreELEC Repo -> Program Add-ons) ]*
- *[ Reboot (necessary before installing the Argon Add-on so that the system tools work properly) ]*
- Upload the ZIP file to /tmp, /storage or another place where you have access via KODI
- Allow installation of Add-ons from external sources:
Enable "Settings->System->Addons->Unknown sources".
- In the main menu within Add-ons select "Install from ZIP file", confirm the security question and switch to the folder where you uploaded the ZIP file
- Select the ZIP file and press OK
- After the first installation: Ignore the “Device Configuration Error” teaser message and reboot to enable the UART, IR and I2C modules

Within Addons list, the ArgonForty Device Configuration add-on should be available now. There you can configure the fan control. The shutdown and reboot (double tab) should work now too. Please be patient, it will take a few seconds for the LED to turn off.

## Integration into the LibreELEC build environment (for Developers)

- Clone the LibreELEC.tv repo

    ```bash
    git clone git@github.com:LibreELEC/LibreELEC.tv.git
    ```

- Change into the LibreELEC.tv directory

    ```bash
    cd LibreELEC.tv
    ```

- Create the add-on package directory

    ```bash
    mkdir -p packages/addons/service/argononecontrol
    ```

- Copy the package.mk into the add-on directory

    ```bash
    wget -O packages/addons/service/argononecontrol/package.mk https://raw.githubusercontent.com/HungerHa/libreelec_addon_argononecontrol/refs/heads/master/package.mk
    ```

- Start the build process

    ```bash
    PROJECT=ARM ARCH=aarch64 DEVICE=ARMv8 scripts/create_addon argononecontrol
    ```
