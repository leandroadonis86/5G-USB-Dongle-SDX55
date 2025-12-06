# 5G-USB-Dongle-SDX55
Fix, Debug, Access, Tools, Tests and more functionalities for this 5G USB Dongle equipped with Qualcomm SDX55 also known as APAL TRIBUTO 5G, COMPAL 5G Dongle, TRI CASCADE VOS 5G.

+ Shell, Linux sdxprairie 4.14.117-perf #1 PREEMPT armv7l GNU/Linux
+ OS release, mdm 202412101203
+ SW, RXMG1.20.00.326.71_0R19; RXMG1.20.00.326.72_0R19 or RXMG1.20.00.326.73.00_0R19 (upgraded)
+ CPU, codename: SDXPRAIRIE, Quectel RM502Q-AE (compatible)
+ Hardware, TRITON SG500M2-X M.2 or Compal GKRRXMG1 M.2 

## Disclaimer

The code and scripts provided herein are for educational purposes only. By using this code, you acknowledge and agree to the following terms:
  1. Use at Your Own Risk: The authors and contributors of this code are not responsible for any damages, losses, or issues that may arise from its use. You use this code at your own risk.
  2. Outdated or Incompatible: The code may be outdated or incompatible with your specific version of software, device, or environment. It is your responsibility to ensure that the code is appropriate for your particular circumstances.
  3. Technical Knowledge Required: This code is intended for individuals with a reasonable level of technical knowledge. You should only use this code if you are confident in your ability to understand and troubleshoot any potential issues.
  4. No Assumption of Responsibility: The authors do not assume any responsibility for any consequences resulting from the use of this code, including but not limited to data loss, system failures, or any other damages.

By proceeding to use this code, you accept and agree to this disclaimer in its entirety.

If you need any information or help just create a "Issue" refering this project with label "Help wanted". I might have more information updated particulary focus in your issue than this general topics readme text.

## 5G USB Dongle
This repository refers to the OEM 5G USB Dongle SDX55 (Fenvi store) from online marketplaces aliexpr... etc, but it can also applied to the known device called "VOS 5G Dongle". The software contains an operative system based on linux (Yocto) with BusyBox and snippets from CodeLinaro. For development modules Qualcomm SDK and OpenQuectel base code may be used.

The hardware is composed with a 5G Sub-6GHz module with M2 form factor with 4 connected external antena and one hardware interface with a single SIM reader, USB-C builtin.


### Electronics

![alt text](https://github.com/leandroadonis86/5G-USB-Dongle-SDX55/blob/main/5Gdongle.png)

#### Inside,

- Nano-SIM card reader, (HD3220)
- M.2 to USB Type C Converter Adapter, SH14 E248779 94V-0; ZX01_GS-419 REV:1C 
- TRITOM SG500M2-X Module, ZX56 GA-597 Rev:1.0

#### Functionalities,

...

### Upgrade
Can be upgraded and drivers are available at tricascadeinc.com.

## Issues

### Vulnerability
For now, I didn't find any issues or cybersecurity report on internet but there are some safety issues: On SSH session device still have builtin manufacture password that can be acessed by external clients with admin\root privileges and have total control of system files\services. This device use cgi-web style that can be a target for code injection.

### Troubleshooting

...

## Access

Access by Modem's webpage in `https://192.168.225.1` login admin/admin equipped with a Web GUI interface, firewall, SIM configuration and other other configuration options.
SSH is accessible and session still have builtin manufacture password that can be acessed by external clients with admin\root privileges for total system control.

There are few open ports in this device used to control flow by internal components: `22/tpc ssh`, `53/tcp domain`, `80/tcp http`, `111/tcp rpcbind`, `443/tcp https`, `7777/tcp qmi_ip_multiclient`.

`Port 111/tcp rpcbind` is responsable of RPC binding according to Qualcomm docs "RPC binding means binding with a remote service for an application. Because the RPC proxy creates a local TelAF proxy service for each RPC service, the TelAF application can leverage the binding section in the .adef file to set the RPC binding." 

`Port 7777/tcp qmi_ip_multiclient` consists in "QMI" (Qualcomm Messaging Interface). It is a proprietary protocol used by Qualcomm chipsets to allow different software components on a device to communicate with each other, particularly between a modem and other subsystems like the host PC. The term "multi-client" refers to the client-server model, where multiple client applications can communicate with a single modem service through the QMI framework, often via a multiplexer daemon like qmuxd. This is used to handle tasks like managing data connections, network access, and device information.

Any other Access can be block or allow with iptables available in the system.


## Recommendations

1. USB-C to USB-C Data cable for 10Gbs only and direct connected.
2. Avoid firmware update it can go wrong in the update proccess.
3. Change as possible admin password at WebGUI.

## Development
One way easy to build an executable to run it on router is using the cross-compile option. Device system doesn't have any gcc toolchain to compile C however it runs files detailed like below:
```
ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, stripped
```

## Tools
- ### Hardware
USB-C to USB-C Data cable for 10Gbs






