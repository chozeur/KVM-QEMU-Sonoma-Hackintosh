This repo aims to guide you threw the setup of a macOS (Sonoma 14.3) virtual machine. We'll deploy it using [KVM](https://www.redhat.com/en/topics/virtualization/what-is-KVM) and [QEMU](https://www.qemu.org) under an [UNRAID](https://unraid.net) server.

# Prerequisites

* A macOS machine to create the installation media

* Check your components compatibility with macOS. You can use [this](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html) guide to check it.

* Patience. This is not a simple process and it will take some time to get it working.

# GPU compatibility

Once your VM will be up and running, you'll need to pass a GPU to it. Since macOS stopped supporting NVIDIA GPUs, I strongly recommend to use an AMD one (some NVIDIA might do the job, like GT 710, but you'll probably face some extra settings that I will not cover here). You can use any AMD GPU, but I suggest using a Polaris or Navi one, since they are natively supported by macOS.

The list of supported GPUs can be found [here](https://dortania.github.io/GPU-Buyers-Guide/modern-gpus/amd-gpu.html).

# UNRAID setup

First of all, you'll need to have an UNRAID server up and running. If you don't have one, you can follow [this](https://docs.unraid.net/unraid-os/getting-started/quick-install-guide/) guide to set it up.

I will not cover the setup of the UNRAID server here, but I'll guide you threw the setup of the macOS VM. For any UNRAID related questions, you can check the [official documentation](https://wiki.unraid.net/Main_Page). I also recommend to join the [UNRAID forums](https://forums.unraid.net) and the [UNRAID subreddit](https://www.reddit.com/r/unRAID/). Overall, here's probably the best YouTube channel to learn about UNRAID: [Spaceinvader One](https://www.youtube.com/c/SpaceinvaderOne).

# Create macOS installation media

* ## Download macOS installer

	* Get [gibMacOS](https://github.com/corpnewt/gibMacOS) : `git clone https://github.com/corpnewt/gibMacOS.git`

	* Run `gibMacOS.command` and select the macOS version you want to download (Sonoma in this case).

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

* ## Download the KVM-OpenCore image

	* On [this](https://github.com/thenickdude/KVM-Opencore) repo, download the `KVM-OpenCore` [image](https://github.com/thenickdude/KVM-Opencore/releases/download/v20/OpenCore-v20.iso.gz)
	* Extract the `.gz` file: `gzip -d OpenCore-v20.iso.gz`
	* Change the name of the `.iso` file to `OpenCore.img`: `mv OpenCore-v20.iso OpenCore.img`
	* Copy the `.img` file to your UNRAID ISOs directory (you can use the same method as for the macOS `.img` file)

# macOS VM setup

* ## Step 1: Create a macOS VM

	* Go to UNRAID web interface, in Apps tab, and search for "macinabox".

	* Set the following settings:
		* **Network Type**: Bridge
		* **Console shell command**: Shell
		* **Privileged**: On
		* **Operating System Version**: Monterey

# Credits

* [u/pirokiki](https://www.reddit.com/user/pirokiki/) for the undefeatable patience, the great help and the so useful [guide](https://www.reddit.com/r/unRAID/comments/17rmf1y/hi_i_managed_to_create_sonoma_vm_on_unraid_with/) on how to install macOS Sonoma on UNRAID
* [Dortania](https://dortania.github.io) for the amazing guides on how to install macOS on non-Apple hardware
* [Spaceinvader One](https://www.youtube.com/c/SpaceinvaderOne) for the amazing UNRAID tutorials
* [i12bretro](https://i12bretro.github.io/tutorials/) for the great tutorials
