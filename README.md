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

* ## Create the bootable `.img` file



# macOS VM setup

* ## Step 1: Create a macOS VM

	* Go to UNRAID web interface, in Apps tab, and search for "macinabox".

	* Set the following settings:
		* **Network Type**: Bridge
		* **Console shell command**: Shell
		* **Privileged**: On
		* **Operating System Version**: Monterey

