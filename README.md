## Banana Pro LIRC GPIO transmitter

Based off https://gist.github.com/zerkalica/f1c3a0a889cd974c495d

I was using Bananian and after reading https://github.com/igorpecovnik/lib/issues/135 I switched to using the Armbian Legacy Jessie kernel (http://www.armbian.com/banana-pi-pro/) to get the kernel module `lirc_gpio`.

### Direct lircd usage


I am using GPIO physical pin 3 (aka sys pin 2, see sys column http://wiki.lemaker.org/How_to_control_the_IO_on_the_SBC_boards#Appendix:_40_Pins_GPIO_Mapping_Table_for_Banana_Pro_and_LeMaker_Guitar for the numbers to use with sysfs). The `lirc_gpio` module must be configured with the output pin you wish to use. This can be configured with kernel module parameters e.g. `modprobe lirc_gpio gpio_out_pin=2` or via sysfs e.g. `echo 2 >/sys/class/lirc_gpio/tx_gpio_pin`. See below:

```
# This resolves to /dev/lirc0 for me
export LIRC_DEVICE=/dev/$(ls /sys/devices/platform/lirc_gpio.0/lirc/)
```

Using kernel module parameter
```
modprobe lirc_gpio gpio_out_pin=2
lircd --device=$LIRC_DEVICE --driver=default remotes.conf
irsend SEND_ONCE LG_AKB72915207 KEY_POWER
```

Using sysfs
```
modprobe lirc_gpio
echo 2 >/sys/class/lirc_gpio/tx_gpio_pin
lircd --device=$LIRC_DEVICE --driver=default remotes.conf
irsend SEND_ONCE LG_AKB72915207 KEY_POWER
```

### lircd dameons

Based off this [gist](https://gist.github.com/zerkalica/f1c3a0a889cd974c495d) by @zerkalica I updated `/etc/init.d/lirc` to run two daemons. One is for the `devinput` receiver device and the other is for the `lirc_gpio` transmitter device.

See [lirc](./lirc) for the updated script and [lirc.diff](./lirc.diff) to see the changes I made. Also have a look at [hardware.conf](./hardware.conf). I've also modified the receiver device to use `/dev/input/cb2ir` after reading http://linux-sunxi.org/LIRC#Using_LIRC_with_Cubieboard2_.28A20_SoC.29 and creating `/etc/udev/rules.d/10-cb2ir.rules` with:

`SUBSYSTEM=="input", ACTION=="add", KERNEL=="event*", ATTRS{name}=="sunxi-ir", SYMLINK+="input/cb2ir"`

Note that this broke for me when I tried to use the `sunxi-lirc` module to get `irrecord` working because the name no longer matched. In any case I haven't needed `irrecord` just yet because I found the remote configs I needed online.

The transmitter daemon sets up a lirc device at `/dev/lircd-tx` that you can use with `irsend`.

```
irsend -d /dev/lircd-tx SEND_ONCE LG_AKB72915207 KEY_POWER
```

The original `lircd` for the onboard IR receiver on `devinput` is available at the normal `/dev/lircd`.

