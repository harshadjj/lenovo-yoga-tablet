# lenovo-yoga-tablet
Linux instructions and configurations for Lenovo Yoga Tablet 2 (1051f) Windows version

#### Things to take note of:
 - Tablet uefi firmware is 32bit but cpu architecture is 64bit

#### Bluetooth
Download brcm and firmware then add this to bluetooth service

    /usr/local/bin/brcm_patchram_plus --no2bytes --baudrate 3000000 --use_baudrate_for_download --patchram /etc/firmware/brcm/BCM4324B3_002.004.006.0130.0161.hcd -bd_addr $MAC /dev/ttyS4 -d >/tmp/btlog 2>&1

> NOTE: In newer linux versions this may not be necessary. Just put firmware to path `/etc/firmware/brcm/BCM4324B3_002.004.006.0130.0161.hcd` and kernel will handle rest.

#### GPU Acceleration for X11
Enable early kms and add below given parameteres to kernel command line.

    i915.modeset=1 intel_pstate=disable intel_idle.max_cstate=1 clocksource=tsc

Then create /etc/X11/xorg.conf.d/20-intel.conf:

    Section "Device"
        Identifier "Intel Graphics"
        Driver "intel"
        Option "DPMS"
        Option "AccelMethod" "sna"
        Option "TearFree" "True"
    EndSection

#### Backlight

Create backlight change script at /usr/local/bin/setBacklight:

    #!/bin/bash

    if [[ "$1" == "off" ]];
    then
        /usr/sbin/i2cset -y 9 0x2c 0x00 0
    elif [[ "$1" == "on" ]];
    then
        /usr/sbin/i2cset -y 9 0x2c 0x00 1
    elif [[ "$1" == "read" ]];
    then
        i2cget -y 9 0x2c 0x04
    else
        /usr/sbin/i2cset -y 9 0x2c 0x00 0
        /usr/sbin/i2cset -y 9 0x2c 0x10 0x85
        /usr/sbin/i2cset -y 9 0x2c 0x04 $1
        /usr/sbin/i2cset -y 9 0x2c 0x00 1
    fi

and /lib/systemd/system-sleep/backlight:

    #!/bin/sh

    if [[ "$1" == "pre" ]];
    then
        /usr/local/bin/setBacklight read > /var/tmp/brightness
        /usr/local/bin/setBacklight off
    else
        /usr/local/bin/setBacklight on
        if [ -f /var/tmp/brightness ]
        then
            /usr/local/bin/setBacklight `cat /var/tmp/brightness`
        fi
        /bin/systemctl restart  bluetooth.service
        /usr/sbin/powertop --auto-tune
    fi
