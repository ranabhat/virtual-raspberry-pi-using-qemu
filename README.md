# RUN Raspberry PI VM using QEMU 
- [RUN Raspberry PI VM using QEMU](#run-raspberry-pi-vm-using-qemu)
  - [Prerequisites:](#prerequisites)
  - [Instructions:](#instructions)
  - [Explanation of the run VM command:](#explanation-of-the-run-vm-command)

## Prerequisites:

- macOS
- Homebrew (for installing QEMU) `brew install qemu`

## Instructions:

The instruction is for Mac OS.

1. Download a Raspberry Pi OS Image

   - Download the latest Raspberry Pi OS image for ARM architecture. The Raspberry Pi OS Lite version is commonly used for QEMU emulation due to its smaller size.
    - Get the image from the official Raspberry Pi website.
    - The os image is not included in the repo, download and extract to the root repo directory after cloning the repo.
  
2. Use the hdiutil command to attach the image file. This will allow you to see the partitions within the image.
   -  `hdiutil attach 2024-10-22-raspios-bookworm-arm64-lite.img`
   -  unmount after the check `hdiutil detach /dev/disk4s1`
  
3. Mount the downloaded image to the `mount-image` folder.
   - `hdiutil attach 2024-10-22-raspios-bookworm-arm64-lite.img -mountpoint mount-image/`
  
4. Go to `mount-image` and copy `bcm2710-rpi-3-b.dtb` and `kernel8.img` to root directory 
   - `cd mount-image`
   - `cp bcm2710-rpi-3-b.dtb kernel8.img ../` 

5. Since raspberry pi can't have default username and password, For [headless setup](https://www.raspberrypi.com/news/raspberry-pi-bullseye-update-april-2022/) create two files `ssh` and `userconf.txt` inside `mount-image`
   -   `touch ssh` 
   -   `touch userconf.txt`
       -   This file should contain a single line of text, consisting of `username:encrypted- password`
           -  password can be generated using `echo 'mypassword' | openssl passwd -6 -stdin`
           -  for test purpose; my username is `tester` and password is `tester`
           -  the text will look like `tester:$6$XJxS8MT6sF6riGVm$7/gJhp.ibeGjp7fNzqacyRSBTI.KgYecB9T5GbyCRwsXKJAGRAVTCRuw0GMuywOn.McSwfFSnrZMmPe5OnALQ0`

6. Umount the disk `hdiutil detach /dev/disk4s1`
7. Make sure copied files are in correct mode
   - `chmod 755 bcm2710-rpi-3-b.dtb` 
   - `chmod 755 kernel8.img` 
8. Resize the image (power of 2, since our image is greater than 2GB)
   - `qemu-img resize -f raw 2024-10-22-raspios-bookworm-arm64-lite.img 4G` 
9.  Run VM (opt1)

   ```bash
    qemu-system-aarch64 \
    -machine raspi3b \
    -cpu cortex-a53 \
    -smp 4 \
    -m 1G \
    -kernel kernel8.img \
    -dtb bcm2710-rpi-3-b.dtb \
    -drive file=2024-10-22-raspios-bookworm-arm64-lite.img,format=raw,if=sd \
    -append "root=/dev/mmcblk0p2 rw rootwait rootfstype=ext4" \
    -usbdevice keyboard \
    -usbdevice mouse \
    -netdev user,id=net0,hostfwd=tcp::2022-:22 \
    -device usb-net,netdev=net0 \
    -nographic
    ```

10. Run VM (opt2)

    ```bash
    qemu-system-aarch64 \
    -machine raspi3b \
    -cpu cortex-a53 \
    -smp 4 \
    -m 1G \
    -kernel kernel8.img \
    -dtb bcm2710-rpi-3-b.dtb \
    -drive file=2024-10-22-raspios-bookworm-arm64-lite.img,format=raw,if=sd \
    -append "rw earlyprintk loglevel=8 console=ttyAMA0,115200 root=/dev/mmcblk0p2 rootdelay=1" \
    -usbdevice keyboard \
    -usbdevice mouse \
    -netdev user,id=net0,hostfwd=tcp::2022-:22 \
    -device usb-net,netdev=net0 \
    -nographic
    ```

11. Kill VM
    - find process id `ps aux | grep qemu`
    - sudo kill <process-id>



## Explanation of the run VM command:

1. -M raspi3b: Emulates the Raspberry Pi 3 Model B.
2. -cpu cortex-a53: Specifies the ARM Cortex-A53 CPU.
3. -smp 4 -m 1G: Allocates 4 cores and 1GB of RAM.
4. -dtb bcm2710-rpi-3-b.dtb: Uses the correct Device Tree Blob for Raspberry Pi 3B.
5. -kernel kernel8.img: Loads the Raspberry Pi OS kernel
6. -drive file=your_image.img,format=raw,if=sd: Specifies the OS image file as an SD card.
7. -append ...: Configures kernel parameters.
8. -netdev user,id=net0,hostfwd=tcp::2222-:22: Forwards SSH from port 2022 on the host to port 22 on the VM.
9. -nographic: Runs without a graphical interface (useful for SSH access).


