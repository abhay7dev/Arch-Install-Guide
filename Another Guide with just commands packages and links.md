For Windows:

* Decrease size of C: partition
* Use Veracrypt for encryption if required
* Intel Rapid Storage Technology (IRST) can be disabled from Intel Optane Memory and Storage Management App (cannot be disabled in bios)
* Disable Fast Boot
* Disable Hibernation (May conflict with Linux)

For Arch Linux:

`timedatectl set-ntp true`
```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Wifi Network Name>
station wlan0 show
exit
```
```

```







