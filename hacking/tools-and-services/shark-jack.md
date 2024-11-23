---
description: >-
  Purchase one here: https://docs.hak5.org/shark-jack. I'm using the Shark Jack
  Battery, not the Shark Jack Cable. They are different.
---

# Shark Jack

**MODES:**

**OFF:** Closest to USB-C port, or "back" position. Also used for "Charging" mode

**ARM:** middle position, "Management" mode

**ATTACK:** Closest to the ethernet port, or "forward" position

## Charging

{% hint style="warning" %}
DO NOT LEAVE DEVICE UNATTENDED AND DO NOT OVER CHARGE THE DEVICE. IT WILL GET WARM SO LET IT COOL PRIOR TO USE JUST TO BE SAFE.
{% endhint %}

* Turn the switch to the `OFF` position and plug it into a USB power source.
* If the jack is blinking blue, it is charging up.
* It is charged when the jack is solid blue.

## ARM Mode

* Set the switch to the middle position.
* Plug the Shark Jack into your computer's ethernet port.
* Additionally, plug the Shark Jack into a power source reliable power source.
* SSH to the Shark Jack: `ssh root@172.16.24.1`
* Default password: `hak5shark`
* You could connect via serial connection as well, but I could not get it working properly with a standard USB-C cable and Pixel 5a.

## Payloads

* Store your payloads here: `/root/payload/library`
  * I recommend cloning the repo, which should work with the `UPDATE_PAYLOADS` command.
* SSH to the device and use the command `LIST` to list the payloads available.
* Use the `ACTIVATE` command to turn on payloads to run next time ATTACK mode executes. Examples from [Hak5's site](https://docs.hak5.org/shark-jack/managing-payloads/the-activate-command):
  * Use a payload from the library: `ACTIVATE recon/nmap`
  * A specific file: `ACTIVATE /tmp/payload.sh`

### Plunder Bug Switch

Use a Plunder Bug to connect a laptop, Shark Jack, and WAN port all at the same time with a USB -> Ethernet adapter. Really cool concept, you should check it out.

* How-To: [https://docs.hak5.org/shark-jack/tips-and-tricks/using-the-shark-jack-with-the-plunder-bug-as-a-simple-switch](https://docs.hak5.org/shark-jack/tips-and-tricks/using-the-shark-jack-with-the-plunder-bug-as-a-simple-switch)
* Purchase Here: [https://shop.hak5.org/products/bug](https://shop.hak5.org/products/bug)

### Development

* Web IDE for crafting payloads: [https://payloadstudio.hak5.org/community/](https://payloadstudio.hak5.org/community/)
* To install a dependancy package, you can run the following command:
  * `opkg install coreutils-timeout zip`
* Basic tutorial: [https://docs.hak5.org/shark-jack/beginner-guides/writing-a-simple-payload](https://docs.hak5.org/shark-jack/beginner-guides/writing-a-simple-payload)
* Payloads: [https://github.com/hak5/sharkjack-payloads](https://github.com/hak5/sharkjack-payloads)

### Firmware Upgrade

{% hint style="warning" %}
PULLING THE DEVICE BEFORE THE UPGRADE COMPLETES WILL LIKELY BRICK YOUR DEVICE. IF YOU DO NOT HAVE A RELIABLE POWER SOURCE YOU MAY BRICK YOUR DEVICE
{% endhint %}

* Download the bin file from here: [https://downloads.hak5.org/shark/battery](https://downloads.hak5.org/shark/battery)
  * I'm using the battery version. If you have the Shark Jack Cable, you'll want to go get that firmware.
* Transfer the `.bin` file: `scp upgrade-1.1.0.bin root@172.16.24.1:/tmp/`
* Default password: `hak5shark`
* SSH to the Shark Jack: `ssh root@172.16.24.1`
* Default password: `hak5shark`

{% hint style="danger" %}
PULLING THE DEVICE BEFORE THE UPGRADE COMPLETES WILL LIKELY BRICK YOUR DEVICE. IF YOU DO NOT HAVE A RELIABLE POWER SOURCE YOU MAY BRICK YOUR DEVICE
{% endhint %}

* Execute: `sysupgrade -n /tmp/upgrade-1.1.0.bin`
* Could take 5-10 minutes to complete, so I suggest not touching it for 15.



