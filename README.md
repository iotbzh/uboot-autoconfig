# Automated [U-Boot] Configuration

The script send_uboot_config is able to push [U-Boot] configs directly to a board connected through a serial console.

## Configuration

Edit the file boards.conf (ini style) and declare a new board. The section name gives the name of the board and all fields are mandatory:

```
[myboard-home-m3]
serialport = /dev/ttyUSB0
baudrate   = 115200

# board type: m3ulcb, h3ulcb
board      = m3ulcb

# SOC number: r8a7796 (M3), r8a7795 (H3)
soc        = r8a7796

# bootmode: net, sd, sdi (sdcard with inetd)
bootmode   = net

# mmc device for sd(i) modes
bootmmc    = 0:1

# board ip address for netboot
ipaddr     = 10.0.0.10

# server ip address for netboot
serverip   = 10.0.0.5

# board MAC address to be set on ethernet interface
macaddr    = DE:AD:C0:FF:EE:00
```

## U-Boot template file

The files \*.ubootconf are templates where @XXX@ variables are replaced by the ones set in the INI section.

## Usage

**Step 1** : start a terminal (minicom, picocom, screen, socat ...)

**Step 2** : start the board and abort boot to get U-Boot prompt

```
U-Boot 2015.04 (Oct 11 2017 - 20:47:11)

CPU: Renesas Electronics R8A7796 rev 1.0
Board: M3ULCB
I2C:   ready
DRAM:  1.9 GiB
MMC:   sh-sdhi: 0, sh-sdhi: 1
In:    serial
Out:   serial
Err:   serial
Net:   ravb
Hit any key to stop autoboot:  0 
=>   
```

**Step 3** : Exit from the terminal

* minicom: Ctrl+a x
* picocom: Ctrl+a Ctrl+x
* screen: Ctrl+a Ctrl+\

**Step 4** : Run ***send_uboot_config***

```
./send_uboot_config <board name> <uboot config>
```

to apply the specified U-Boot config to the named board.


Sample output:
```
./send_uboot_config sdome-m3 m3ulcb-kf.ubootconf
board: m3ulcb
soc: r8a7796
bootmode: net
bootmmc: 0:1
ipaddr: 10.0.0.29
serverip: 10.0.0.15
macaddr: 2E:0A:0A:00:87:BD
Configuring serial port /dev/ttyUSB1 with speed 115200
env default -a
## Resetting to default environment
=> setenv board m3ulcb
=> setenv soc r8a7796
=> setenv bootmode net
=> setenv bootmmc '0:1'
=> setenv ipaddr '10.0.0.29'
=> setenv serverip '10.0.0.15'
=> setenv ethaddr '2E:0A:0A:00:87:BD'
=> setenv ethact ravb
=> setenv set_bootkfile 'setenv bootkfile Image'
=> setenv bootkaddr 0x48080000
=> setenv set_bootdfile 'setenv bootdfile Image-${soc}-${board}-kf.dtb'
=> setenv bootdaddr 0x48000000
=> setenv set_bootifile 'setenv bootifile initramfs-netboot-image-${board}.ext4.gz'
=> setenv bootiaddr 0x5C3F9520
=> setenv bootisize 3A6AB6
=> setenv bootargs_console 'console=ttySC0,115200 ignore_loglevel'
=> setenv bootargs_video 'vmalloc=384M video=HDMI-A-1:1920x1080-32@60'
=> setenv bootargs_extra 'rw rootfstype=ext4 rootwait rootdelay=2'
=> setenv bootcmd 'run bootcmd_${bootmode}'
=> setenv bootkload_sd 'ext4load mmc ${bootmmc} ${bootkaddr} boot/${bootkfile}'
=> setenv bootiload_sd 'ext4load mmc ${bootmmc} ${bootiaddr} boot/${bootifile}'
=> setenv bootdload_sd 'ext4load mmc ${bootmmc} ${bootdaddr} boot/${bootdfile}'
=> setenv bootargs_root_sd 'root=/dev/mmcblk1p1'
=> setenv bootload_sd 'run set_bootkfile; run bootkload_sd; run set_bootdfile; run bootdload_sd'
=> setenv bootcmd_sd 'setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_sd} ${bootargs_extra}; run bootload_sd; booti ${bootkaddr} - ${bootdaddr}'
=> setenv bootargs_root_sdi 'root=/dev/ram0 ramdisk_size=16384'
=> setenv bootload_sdi 'run set_bootkfile; run bootkload_sd; run set_bootdfile; run bootdload_sd; run set_bootifile; run bootiload_sd'
=> setenv bootcmd_sdi 'setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_sdi} ${bootargs_extra}; run bootload_sdi; booti ${bootkaddr} ${bootiaddr}:${bootisize} ${bootdaddr}'
=> setenv bootkload_net 'tftp ${bootkaddr} ${board}/${bootkfile}'
=> setenv bootdload_net 'tftp ${bootdaddr} ${board}/${bootdfile}'
=> setenv bootiload_net 'tftp ${bootiaddr} ${board}/${bootifile}'
=> setenv bootargs_root_net 'root=/dev/ram0 ramdisk_size=16384 ip=dhcp'
=> setenv bootload_net 'run set_bootkfile; run bootkload_net; run set_bootdfile; run bootdload_net; run set_bootifile; run bootiload_net'
=> setenv bootcmd_net 'setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_net} ${bootargs_extra} nbd.server=${serverip}; run bootload_net; booti ${bootkaddr} ${bootiaddr}:${bootisize} ${bootdaddr}'
=> env print
baudrate=115200
board=m3ulcb
bootargs=root=/dev/nfs rw nfsroot=192.168.0.1:/export/rfs ip=192.168.0.20
bootargs_console=console=ttySC0,115200 ignore_loglevel
bootargs_extra=rw rootfstype=ext4 rootwait rootdelay=2
bootargs_root_net=root=/dev/ram0 ramdisk_size=16384 ip=dhcp
bootargs_root_sd=root=/dev/mmcblk1p1
bootargs_root_sdi=root=/dev/ram0 ramdisk_size=16384
bootargs_video=vmalloc=384M video=HDMI-A-1:1920x1080-32@60
bootcmd=run bootcmd_${bootmode}
bootcmd_net=setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_net} ${bootargs_extra} nbd.server=${serverip}; run bootload_net; booti ${bootkaddr} ${bootiaddr}:${bootisize} ${bootdaddr}
bootcmd_sd=setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_sd} ${bootargs_extra}; run bootload_sd; booti ${bootkaddr} - ${bootdaddr}
bootcmd_sdi=setenv bootargs ${bootargs_console} ${bootargs_video} ${bootargs_root_sdi} ${bootargs_extra}; run bootload_sdi; booti ${bootkaddr} ${bootiaddr}:${bootisize} ${bootdaddr}
bootdaddr=0x48000000
bootdelay=3
bootdload_net=tftp ${bootdaddr} ${board}/${bootdfile}
bootdload_sd=ext4load mmc ${bootmmc} ${bootdaddr} boot/${bootdfile}
bootiaddr=0x5C3F9520
bootiload_net=tftp ${bootiaddr} ${board}/${bootifile}
bootiload_sd=ext4load mmc ${bootmmc} ${bootiaddr} boot/${bootifile}
bootisize=3A6AB6
bootkaddr=0x48080000
bootkload_net=tftp ${bootkaddr} ${board}/${bootkfile}
bootkload_sd=ext4load mmc ${bootmmc} ${bootkaddr} boot/${bootkfile}
bootload_net=run set_bootkfile; run bootkload_net; run set_bootdfile; run bootdload_net; run set_bootifile; run bootiload_net
bootload_sd=run set_bootkfile; run bootkload_sd; run set_bootdfile; run bootdload_sd
bootload_sdi=run set_bootkfile; run bootkload_sd; run set_bootdfile; run bootdload_sd; run set_bootifile; run bootiload_sd
bootmmc=0:1
bootmode=net
ethact=ravb
ethaddr=2E:0A:0A:00:87:BD
fdt_high=0xffffffffffffffff
initrd_high=0xffffffffffffffff
ipaddr=10.0.0.29
serverip=10.0.0.15
set_bootdfile=setenv bootdfile Image-${soc}-${board}-kf.dtb
set_bootifile=setenv bootifile initramfs-netboot-image-${board}.ext4.gz
set_bootkfile=setenv bootkfile Image
soc=r8a7796

Environment size: 2211/131068 bytes
```

**Step 5**: reconnect from Terminal and boot or save the config

Boot with temp config:
```
=> run bootcmd
```

To save the config in MMC memory:
```
=> saveenv 
Saving Environment to MMC...
Writing to MMC(1)... done
```

## Future enhancements

Many things could be done:
* rewrite the thing in python (parsing ini files in bash is... well... arguable. But initially, I didn't think I'd have to do that! Not my fault :))
* be smarter in templating,
* automatically solve IP addresses from hostnames,
* have templates fragments to unify things (ex: have the same config for M3 or M3+KF, and switch with a single variable)i
* ... 

The base mechanism is there. Improvements and contributions are welcome.

[U-Boot]:https://www.denx.de/wiki/U-Boot
