# What are we dealing with?
## MacBookPro12,1
- i5-5287U
- 16GB RAM
- 120GB SSD drive

## Operating system of choice:
Fedora 43 pre-release, Gnome 49

# What software did I use?
### TLP - using this [config](tlp.conf). 
Important notes: set it to use deep sleep both on AC and BAT, if you are using the original Broadcom adapter, otherwise you will need to restart the brcmfmac_wcc driver after sleep.

### Throttled - using this [config](throttled.conf). 
The primary purpose is to undervolt the cpu. If you can live with lover performance, feel free to decrease TDP, but in my case the CPU/iGPU performance already feels like it is on the limit when using Gnome.

### Mbpfan - using this [config](mbpfan.conf).
The values weren't super tested or tuned. The primary purpose is to makes sure that the fan starts spinning when the CPU heats up. The BIOS cannot quite take care of it in a good way.

### Service for disabling USB and Lid as wake up calls 
[suspend-fix.service](suspend-fix.service)

By default, laptop will not be able to enter deep sleep, if the laptop goes to sleep with lid open. 

- LID0 - ACPI call for lid
- XHC1 - ACPI call for USB (such as keyboard).

The laptop can still be woken up with a power button.
[A link for more information](https://askubuntu.com/a/1203159)

### FacetimeHD driver
[Firmware](https://github.com/patjak/facetimehd/wiki/Get-Started#firmware-extraction)

[Kmod packaging](https://discussion.fedoraproject.org/t/mulderje-intel-mac-rpms/130045)

This was a finicky process. At first, after installing the firmware and facetimehd-kmod, the kernel was not detecting any peripherials. I had to:
- boot to an older verion of the kernel, uninstall facetimehd-kmod
- install facetimehd-kmod again
- remove akmod-facetimehd kmod-facetimehd-6.17.0-0.rc7.56.fc43.x86_64
- Regenerate dracut `dracut --regenerate-all force`
- install facetimehd-kmod one last time

And then it worked.

### Service for unloading of the facetimeHD driver before sleep and realoading it afterwards
[facetimehd-reload.service](facetimehd-reload.service)
Otherwise the comuper will not wake up from sleep.

### A patch for setting custom battery charge level
[applesmc-next](https://github.com/c---/applesmc-next)

Especially useful with [Battery Health Charging](https://github.com/maniacx/Battery-Health-Charging/) Gnome extension.

### Disabling mitigations
`sudo grubby --update-kernel=ALL --args="mitigations=off"`
It should increase performace by few % - at the cost of increased vulnerability of the system. It comes with no responsibility from my side.

## How does it run?
It took me several days to find all the tiny things preventing this laptop from running well. I would say that the performance is acceptable. The thing which stands out is the 2,5k screen and decent speakers in a laptop with a nice build quality, for - nowadays - a very affordable price. But GNOME looses some frames here and there.

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
