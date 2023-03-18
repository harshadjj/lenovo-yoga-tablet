# lenovo-yoga-tablet
Linux instructions and configurations for Lenovo Yoga Tablet 2 (1051f) Windows version

#### Things to take note of:
 - Tablet uefi firmware is 32bit but cpu architecture is 64bit

#### Bluetooth
Download brcm and firmware then add this to bluetooth service
    /usr/local/bin/brcm_patchram_plus --no2bytes --baudrate 3000000 --use_baudrate_for_download --patchram /etc/firmware/brcm/BCM4324B3_002.004.006.0130.0161.hcd -bd_addr $MAC /dev/ttyS4 -d >/tmp/btlog 2>&1
