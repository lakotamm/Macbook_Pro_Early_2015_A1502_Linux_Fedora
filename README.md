# What are we dealing with?
## MacBookPro12,1
- i5-5287U
- 16GB RAM
- Samsung 980 500GB SSD
- Intel AX210 Wifi module 

I chose to install a different SSD, because with the original Apple one, the CPU could not enter C6 and C7 states. This is supposedly because it did not support Link power management.

And I went with an AX210 card with an adapter becasue the Broadcom module never really worked well.

## Operating system of choice:
Fedora 44, Gnome 50

It is tested with kernel 7.1.8

# What software did I use?
### TLP - using this [config](tlp.conf). 
Important notes: set it to use deep sleep both on AC and BAT, if you are using the original Broadcom adapter, otherwise you will need to restart the brcmfmac_wcc driver after sleep.

Modes:

Performance - Performance Governor, power saving disabled, will not enter C6/C7 states

Balanced - Conservative governor, it has all energy saving settings turned on, it will enter C6/C7 states

Power Saver - Same as Balanced, but it also has GPU and CPU frequency restrictions

### Throttled - using this [config](throttled.conf). 
The primary purpose is to undervolt the cpu. If you can live with lover performance, feel free to decrease TDP, but in my case the CPU/iGPU performance already feels like it is on the limit when using Gnome.

### Mbpfan - using this [config](mbpfan.conf).
The values weren't super tested or tuned. The primary purpose is to makes sure that the fan starts spinning when the CPU heats up. The BIOS cannot quite take care of it in a good way.

### Service for disabling USB and Lid as wake up calls 
[suspend-fix.service](suspend-fix.service)

By default, laptop will not be able to enter deep sleep, if the laptop goes to sleep with lid open. Disable Lid switch as a wake up call

- LID0 - ACPI call for lid

Place it into: `/etc/systemd/system/suspend-fix.service`

And enable using systemctl. The laptop can still be woken up with a power button.
[A link for more information](https://askubuntu.com/a/1203159)

### FacetimeHD driver, switching on/off the camera and power saving of the camera (ASPM)
[lakotamm/facetimehd-toggle](https://github.com/lakotamm/facetimehd-toggle)

I have set up a fork of facetimehd-toggle, and integrated into it both switching of the camera and ASPM - power saving options. These are important to set up well if you want to achieve C6/C7 idle states of the CPU

This app will:
- disable webcam during startup and before sleep
- allow you to manually turn it on
- automatically enable/disable ASPM/power saving of the webcam and PCIE bus.

### A patch for setting custom battery charge level
[applesmc-next](https://github.com/netlinux-ai/applesmc-next)


Use this fork from netlinux-ai to make it work with kernel 7.1+

Especially useful with [Battery Health Charging](https://github.com/maniacx/Battery-Health-Charging/) Gnome extension.

### Disabling mitigations
`sudo grubby --update-kernel=ALL --args="mitigations=off"`

It should increase performace by few % - at the cost of increased vulnerability of the system. It comes with no responsibility from my side.

### Getting H.264/AVC1 hardware decode support
For some reason, the `libva-intel-media-driver` driver which Fedora comes with does not support H.264 decode on the iGPU.
The solution which I came up with is to replace the driver with `libva-intel-driver` and install `libavcodec-freeworld` from RPM Fusion repository


[Configure RPM fusion](https://rpmfusion.org/Configuration)

`sudo dnf remove libva-intel-media-driver`

`sudo dnf install libva-intel-driver`

`sudo dnf install libavcodec-freeworld`

Source: [Fedora project](https://fedoraproject.org/wiki/Firefox_Hardware_acceleration#Configure_VA-API_Video_decoding_on_Intel)

### An Easy Effects profile for built-in speakers 
[Speakers.json](Speakers.json)

Probably not the best one, but will do it better than no equalizer.

[Easy Effects](https://github.com/wwmm/easyeffects)


## How does it run?
It took me several weeks to find all the tiny things preventing this laptop from running well. I would say that the performance is acceptable. The thing which stands out is the 2,5k screen and decent speakers in a laptop with a nice build quality, for - nowadays - a very affordable price. But since GNOME looses some frames here and there, I simply set it to 1080p resolution. This also helps improve battery life - which is otherwise good, but not when the GPU boosts to keep up with GNOME in 2.5k. With 1080p it is fine.

### Power consumption - with new SSD, AX210 and CPU reaching C7 states
- Office work and writing text - around 8W
- Working with music - around 10W
- Idle with minimum light - around 5,5W

### Power consumption - with the original SSD and Broadcom WLAN
- Office work and writing text - around 9W
- Working with music - around 11W
- Idle with minimum light - around 7,5W
