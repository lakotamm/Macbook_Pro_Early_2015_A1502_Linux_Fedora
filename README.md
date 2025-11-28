# What are we dealing with?
## MacBookPro12,1
- i5-5287U
- 16GB RAM
- Samsung 980 500GB SSD
- Intel AX210 Wifi module

## Operating system of choice:
Fedora 43, Gnome 49

# What software did I use?
### TLP - using this [config](tlp.conf). 
Important notes: set it to use deep sleep both on AC and BAT, if you are using the original Broadcom adapter, otherwise you will need to restart the brcmfmac_wcc driver after sleep.

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

### FacetimeHD driver
[Firmware](https://github.com/patjak/facetimehd/wiki/Get-Started#firmware-extraction)

[Kmod packaging](https://discussion.fedoraproject.org/t/mulderje-intel-mac-rpms/130045)

The first time it was a finicky process. At first, after installing the firmware and facetimehd-kmod, the kernel was not detecting any peripherials. I had to:
- boot to an older verion of the kernel, uninstall facetimehd-kmod
- install facetimehd-kmod again
- remove akmod-facetimehd kmod-facetimehd-6.17.0-0.rc7.56.fc43.x86_64
- Regenerate dracut `dracut --regenerate-all force`
- install facetimehd-kmod one last time

And then it worked.

Second time it somehow worked straight after installation

### An app for manual toggling of FacetimeHD driver
[facetimehd-toggle](https://github.com/Chamal1120/facetimehd-toggle)

Having the FacetimeHD driver enabled means that the CPU cannot enter C6 and C7 power states. Solution?

Blacklist it on startup and enable it only on demand using facetimehd-toggle.

### Service for automatically unloading of the facetimeHD driver before sleep
[facetimehd-unload.service](facetimehd-unload.service)

If you try to put the laptop to sleep with FacetimeHD driver loaded, you will realize that it never loads back. That is an issue. This service fixes it, because it automatically onloads the driver whenever the laptop goes to sleep.

Place it into: `/etc/systemd/system/facetimehd-unload.service` and enable.

### Service for enabling power-saving on camera module
[facetimehd_aspm-tuning.service](facetimehd_aspm-tuning.service)

By default, if the FacetimeHD driver is at any point enabled, the camera module will not enter power-saving state - even if the driver is unloaded, and as a result of that the CPU will not be able to enter C6 and C7 states. This service enables ASPM on the camera module and fixes the issue.

Place it into: `/etc/systemd/system/facetimehd_aspm-tuning.service`

And enable using systemctl.

### Service disabling CPU cores before sleep and re-enabling them afterwards. 
[cpu_sleep.service](cpu_sleep.service)
[cpu_wake.service](cpu_wake.service)
Disabling CPU cores before sleep will speed up a LOT wake up time from sleep. From 7s to 2s.

Place them into: `/etc/systemd/system/cpu_sleep.service` and `/etc/systemd/system/cpu_wake.service`

And enable using systemctl. 

### A patch for setting custom battery charge level
[applesmc-next](https://github.com/c---/applesmc-next)

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


## How does it run?
It took me several days to find all the tiny things preventing this laptop from running well. I would say that the performance is acceptable. The thing which stands out is the 2,5k screen and decent speakers in a laptop with a nice build quality, for - nowadays - a very affordable price. But GNOME looses some frames here and there and Youtube is limited to 1080p with H.264.

### Power consumption
- Office work and writing text - around 9W
- Working with music - around 11W
- Idle with minimum light - around 7,5W.

This is primarily limited by the CPU package, which cannot go below the C3 state, due to the SATA drive/driver not supporting SATA link power management.

Without a SATA drive the CPU package can enter C6 state and it is possible to reach ca 6,5-6,7W consumption. 

Disabling the Broadcom driver and relying on a more modern Wireless adapter allows a further decrease to around 6W.

## Next steps:
- Upgrade the wifi card using an adapter to AX210
- Upgrade the SSD to NVME
