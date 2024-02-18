This repo aims to guide you threw the setup of a macOS (Sonoma 14.3) virtual machine. We'll deploy it using [KVM](https://www.redhat.com/en/topics/virtualization/what-is-KVM) and [QEMU](https://www.qemu.org) under an [UNRAID](https://unraid.net) server.

# Disclaimer

This guide is rendered for educational purposes only. </br>
I am not affiliated with Apple Inc. or any of its subsidiaries or affiliates in any way. </br>
Apple, Mac, macOS, and other Apple-related terms are registered trademarks of Apple Inc. All other trademarks, service marks, product names, and company names or logos mentioned on this guide are the property of their respective owners. </br>
The official Apple website can be found at https://www.apple.com </br>
It’s advisable to consult with legal counsel before proceeding further.

# Prerequisites

* A macOS machine to create the installation media

* Check your components compatibility with macOS. You can use [this](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html) guide to check it.

* I strongly recommend to use a dedicated PCIe USB controller to pass USB devices to the VM. I didn't manage to pass any of mine without it. [This](https://amzn.to/3wi4Stb) is the one I use. It worked out of the box.

* Patience. This is not a simple process and it will take some time to get it working.

# GPU compatibility

Once your VM will be up and running, you'll need to pass a GPU to it. Since macOS stopped supporting NVIDIA GPUs, I strongly recommend to use an AMD one (some NVIDIA might do the job, like GT 710, but you'll probably face the need of some extra settings that I will not cover here). You can use any AMD GPU, but I suggest using a Polaris or Navi one, since they are natively supported by macOS.

The list of supported GPUs can be found [here](https://dortania.github.io/GPU-Buyers-Guide/modern-gpus/amd-gpu.html).

# UNRAID setup

First of all, you'll need to have an UNRAID server up and running. If you don't have one, you can follow [this](https://docs.unraid.net/unraid-os/getting-started/quick-install-guide/) guide to set it up.

I will not cover the setup of the UNRAID server here. For any UNRAID related questions, you can check the [official documentation](https://wiki.unraid.net/Main_Page). I also recommend to join the [UNRAID forums](https://forums.unraid.net) and the [UNRAID subreddit](https://www.reddit.com/r/unRAID/). Above all, here's probably the best YouTube channel to learn about UNRAID: [Spaceinvader One](https://www.youtube.com/c/SpaceinvaderOne).

You'll need to have the [User Scripts](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/) and [vm_custom_icons](https://github.com/SpaceinvaderOne/unraid_vm_icons) plugins installed

# Create macOS installation media

* ## Download macOS installer

	* Get [gibMacOS](https://github.com/corpnewt/gibMacOS) : `git clone https://github.com/corpnewt/gibMacOS.git`

	* In the cloned repo, run `gibMacOS.command` and select the macOS version you want to download (Sonoma in this case).

	* Run the downloaded `InstallAssistant.pkg` and follow the instructions.

* ## Create the bootable `.img` file

	* On a macOS machine, open Disk Utility
	* Click `File` > `New Image` > `Blank Image`
	* Set the following settings:
		* **Save as**: macOS
		* **Name**: macOS
		* **Size**: 16384 Mo
		* **Format**: Mac OS Extended (Journaled)
	* Click `Save`
	* In terminal, run `sudo /Applications/Install\ macOS\ Sonoma.app/Contents/Resources/createinstallmedia --volume /Volumes/macOS --nointeraction`
	* Once the process is done, run: `hdutil detach --force /Volumes/Install\ macOS\ Sonoma`
	* Convert the disk image to a `.cdr` file: `hdiutil convert /PATH/TO/macOS.dmg -format UDTO -o ~/Desktop/macOS.cdr` (replace `/PATH/TO/macOS.dmg` with the path to your `.dmg` file)
	* Rename the `.cdr` file to `.img`: `mv ~/Desktop/macOS.cdr ~/Desktop/macOS.img`

* ## Copy the `.img` file to UNRAID

	There are many ways to do this, here is how to do it using a Python http server

	* On your macOS machine, run `python3 -m http.server 8000` in the directory where the `.img` file is located
	* Open an SSH session to your UNRAID server, or use the terminal in the web interface
	* Run `curl -o /PATH/TO/YOUR/UNRAID/ISOS/DIRECTORY http://YOUR_MACOS_IP:8000/macOS.img` (replace `YOUR_MACOS_IP` with your macOS machine IP and `/PATH/TO/YOUR/UNRAID/ISOS/DIRECTORY` with the path to your UNRAID ISOs directory)

# Get KVM-OpenCore image

* On [this](https://github.com/thenickdude/KVM-Opencore) repo, [download](https://github.com/thenickdude/KVM-Opencore/releases/download/v20/OpenCore-v20.iso.gz) the `KVM-OpenCore` image
* Extract the `.gz` file: `gzip -d OpenCore-v20.iso.gz`
* Change the name of the `.iso` file to `OpenCore.img`: `mv OpenCore-v20.iso OpenCore.img`
* Copy the `.img` file to your UNRAID ISOs directory (you can use the same method as for the macOS `.img` file)

# macOS VM setup

* ## Create a macOS VM

	* Go to UNRAID web interface, in `APPS` tab, and search for "macinabox".

	* Set the following settings:
		* <u>**Network Type**</u> : Bridge
		* <u>**Console shell command**</u> : Shell
		* <u>**Privileged**</u> : On
		* <u>**Operating System Version**</u> : Monterey
		* <u>**Install Type**</u> : Auto install
		* <u>**Vdisk Size**</u> : 100G *(At least)*
		* <u>**Vdisk Type**</u> : raw
		* <u>**Opencore stock or custom**</u> : stock
		* <u>**Delete and replace Opencore**</u> : no
		* <u>**Override default NIC type**</u> : vmxnet3
		* <u>**VM Images Location**</u> : *PATH TO YOUR VMs LOCATION ON UNRAID (if you left it as default, it should be in `/mnt/user/domains`)*
		* <u>**Isos Share Location**</u> : *PATH TO YOUR ISOs LOCATION ON UNRAID (if you left it as default, it should be in `/mnt/user/isos`)*
		* <u>**appdata**</u> : *`$(PATH TO YOUR APPDATA LOCATION ON UNRAID)/macinabox` (if you left it as default, it should be in `/mnt/user/appdata/macinabox`)*

	* Click `Apply`

	* On UNRAID web interface, go to `Settings` tab > `User Scripts` and run in background `1_macinabox_vmready_notify`

	* Once the notification appears, run `1_macinabox_helper`

	* Once the process is done, go to `VMS` tab. **DO NOT START THE VM YET**.

* ## Edit the VM's XML

	The `1_macinabox_helper` script created the VM. It also fixed its XML. From now on, we'll <u>NEVER USE IT AGAIN</u>, since it will overwrite the XML with the wrong settings. Also, we will <u>NEVER USE THE GUI TO EDIT THE VM's SETTINGS</u>, since it will also overwrite the XML with the wrong settings.

	So, to edit the VM's settings, we'll need to edit the XML directly. In the `VMS` tab, click on the VM's icon, then click `Edit`. In top right corner, click `XML VIEW`.

	For now, make the following changes <u>**ONLY**</u> :
	* Change path to the Opencore image to where you stored the one you downloaded [earlier](#get-kvm-opencore-image)
	* Change path to the macOS image to where you stored the one you downloaded [earlier](#copy-the-img-file-to-unraid)
	* Depending on your CPU, add the [according]() qemu command line arguments. I am using an Intel® Core™ i7-10700K, so I added the following arguments *(add it at the end of the `<domain>` section)*:
		```xml
		<qemu:commandline>
			<qemu:arg value='-usb'/>
			<qemu:arg value='-device'/>
			<qemu:arg value='usb-kbd,bus=usb-bus.0'/>
			<qemu:arg value='-device'/>
			<qemu:arg value='isa-applesmc,osk=ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc'/>
			<qemu:arg value='-smbios'/>
			<qemu:arg value='type=2'/>
			<qemu:arg value='-global'/>
			<qemu:arg value='nec-usb-xhci.msi=off'/>
			<qemu:arg value='-global'/>
			<qemu:arg value='ICH9-LPC.acpi-pci-hotplug-with-bridge-support=off'/>
			<qemu:arg value='-cpu'/>
			<qemu:arg value='host,kvm=on,vendor=GenuineIntel,vmware-cpuid-freq=on,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc,+pcid,+ssse3,+sse4.2,+popcnt,+avx,+avx2,+aes,+fma,+bmi1,+bmi2,+xsave,+xsaveopt,+rdrand,check'/>
		</qemu:commandline>
		```

# Credits

* [u/pirokiki](https://www.reddit.com/user/pirokiki/) for the undefeatable patience, the great help and the so useful [guide](https://www.reddit.com/r/unRAID/comments/17rmf1y/hi_i_managed_to_create_sonoma_vm_on_unraid_with/) on how to install macOS Sonoma on UNRAID
* [Dortania](https://dortania.github.io) for the amazing guides on how to install macOS on non-Apple hardware
* [Spaceinvader One](https://www.youtube.com/c/SpaceinvaderOne) for the amazing UNRAID tutorials
* [i12bretro](https://i12bretro.github.io/tutorials/) for the great tutorials
