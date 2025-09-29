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
LID0 - ACPI call for lid
XHC1 - ACPI call for USB (such as keyboard).
The laptop can still be woken up with a power button.
[A link for more information](https://askubuntu.com/a/1203159)

### FacetimeHD driver
[Firmware](https://github.com/patjak/facetimehd/wiki/Get-Started#firmware-extraction)
[Kmod packaging](https://discussion.fedoraproject.org/t/mulderje-intel-mac-rpms/130045)
This was a finicky process, at first leading kernel not detecting any peripherials and me somehow - not sure how - managing to fix it by reinstalling it. If I ever find out how. I will update the repo.

### Service for unloading of the facetimeHD driver before sleep and realoading it afterwards
[facetimehd-reload.service](facetimehd-reload.service)
Otherwise the comuper will not wake up from sleep.

### A patch for setting custom battery charge level
[applesmc-next](https://github.com/c---/applesmc-next)

Especially useful with [Battery Health Charging](https://github.com/maniacx/Battery-Health-Charging/) Gnome extension.
