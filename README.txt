====================== REQUIREMENTS =========================
-Xen binary:
    -compile a xen code, then launch:
        mkimage -A riscv -C none -T kernel -a 0x84000000 -e 0x84000000 -n "XEN" -d xen xen.uImage
-Device tree blob:
    -generate your device tree .dtb file starting from the .dts source file by launching:
        dtc -I dts -O dtb -i <your_source_file>.dts -o eic7700-hifive-premier-p550.dtb
-Linux kernel Image.gz (only compressed archive) for domU
-Linux ramdisk initrd.img for domU

#===================== U-boot setup =========================
How to set a variable

-setenv <variable_name> '<variable_field>' #(i.e. setenv bootcmd 'bootflow scan')

-saveenv #to save the new environment to SPIFlash

Default p550 bootcmd variable:
    bootcmd='bootflow scan'

TFTP variables:
    serverip=<tftp_server_ip>
    ipaddr=<board_ip>
    netmask=<tftp_server_mask>

set your boot.txt with your command to match the original flow, i.e:

    tftpboot 0x84000000 <your_path_to>/xen.uImage #to load xen binary
    tftpboot 0x88000000 <your_path_to>/eic7700-hifive-premier-p550.dtb #to load Device Tree Blob
    tftpboot 0x808ef000 <your_path_to>/Image.gz #to load Linux kernel for domU
    tftpboot 0x90400000 <your_path_to>/initrd.img #to load Linux ramdisk

    bootm 0x84000000 - 0x88000000 #to boot the board with the new loaded binaries
    
    (please note that the specified addresses need to match the devices of the used dtb)

launch the command below to generate your boot.scr script starting from boot.txt:
    mkimage -A riscv -T script -C none -n "OpenEmbedded-Sifive-hifive-premier-p550" -d boot.txt boot.scr
    
Set the new environment to execute the generated script by stopping the autoboot and setting bootcmd variable as below:

    setenv bootcmd 'tftpboot 0x99400000 <your_path_to>/boot.scr; source 0x99400000'
    
Save the environment and reboot as described in section "Run"
    
#========================== Run =============================
Launch a terminal and type:
    picocom -b 115200 /dev/ttyUSB3 # to communicate with MCU
Main commands:
    sompower-s 0 # to turn off the board
    sompower-s 1 # to turn on the board
    reboot warm # to warm reset the board
    reboot cold # to cold reset the board
    
Open another terminal and type:
    screen -L -Logfile <logfile> #to save the output of the log
then:
    screen /dev/ttyUSB2 115200 # to communicate with serial for both input/output
