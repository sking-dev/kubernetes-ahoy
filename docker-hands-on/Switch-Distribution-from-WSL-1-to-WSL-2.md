# Switch from Using WSL 1 to WSL 2

If I remember rightly, I installed WSL 2 as the default on my Windows 10 workstation "back in the day", but I couldn't get it to run an Ubuntu 20.04 distribution so I had to follow the workaround of running the distribution under WSL 1.

The plus side of this is that I already have WSL 2 installed on my system so it should (hopefully!) be a case of, updating anything that needs updating and then switching my existing Linux distribution over to WSL 2.

I'm going to follow this article <https://dev.to/adityakanekar/upgrading-from-wsl1-to-wsl2-1fl9> and I'll summarise the steps below.

## Check Current Distribution and WSL Version

```powershell
PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Running         1   
```

## Set Default Version to WSL 2

```powershell
PS D:\> wsl --set-default-version 2

For information on key differences with WSL 2 please visit https://aka.ms/wsl2
The operation completed successfully.
```

## Convert Current Distribution from WSL 1 to WSL 2

I'm going to try converting from WSL 1 to WSL 2 to see if works without me having to do anything (wishful thinking, I suspect..!)

```powershell
wsl --set-version Ubuntu-20.04 2

Conversion in progress, this may take a few minutes...
For information on key differences with WSL 2 please visit https://aka.ms/wsl2

# This means the conversion hasn't worked - boo!
Please enable the Virtual Machine Platform Windows feature and ensure virtualization is enabled in the BIOS.
For information please visit https://aka.ms/wsl2-install
```

Right, time to investigate why I'm getting this result.  See below for troubleshooting.

----

## Conversion from WSL 1 to WSL 2 Not Working

Here's a recap of where I've got to so far.

```powershell
PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         1

PS D:\> wsl --set-version Ubuntu-20.04 2

Conversion in progress, this may take a few minutes...
For information on key differences with WSL 2 please visit https://aka.ms/wsl2
Please enable the Virtual Machine Platform Windows feature and ensure virtualization is enabled in the BIOS.
For information please visit https://aka.ms/wsl2-install

PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         1
```

### Confirm Necessary Windows Features Enabled

The following features should be enabled except where indicated.

- Hyper-V
  - Hyper-V Management Tools DISABLED
  - Hyper-V Platform
    - Hyper-V Hypervisor DISABLED - THIS IS GREYED-OUT ON MY SYSTEM
    - Hyper-V Services
- Virtual Machine Platform
- Windows Subsystem for Linux

Also try the command below as mentioned in this GitHub issue: <https://github.com/microsoft/WSL/issues/5363>.

```powershell
PS D:\> bcdedit /set hypervisorlaunchtype auto
The operation completed successfully.

# Now to repeat the earlier process to convert my WSL 1 distro to WSL 2.

PS D:\> wsl --status
Default Distribution: Ubuntu-20.04
Default Version: 2

Windows Subsystem for Linux was last updated on 23/06/2022
WSL automatic updates are on.

Kernel version: 5.10.102.1

PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         1

PS D:\> wsl --set-version Ubuntu-20.04 2
Conversion in progress, this may take a few minutes...
For information on key differences with WSL 2 please visit https://aka.ms/wsl2
Please enable the Virtual Machine Platform Windows feature and ensure virtualization is enabled in the BIOS.
For information please visit https://aka.ms/wsl2-install

# This is the same error / unsuccessful outcome as seen previously.
```

From my research, I think there may be some kind of issue at work which prevents the conversion for an Ubuntu 20.04 distribution, and this may potentially be triggered - or compounded - by attempting to run WSL 2 on a VMware-hosted virtual machine (which, I probably should have mentioned earlier, is what I'm doing...)

### Troubleshooting Sources

- <https://askubuntu.com/questions/1398225/unable-to-convert-vmware-ubuntu-wsl1-to-wsl2-because-virtualization-is-not-enab>
- <https://www.tenforums.com/tutorials/164301-how-update-wsl-wsl-2-windows-10-a.html>
- <https://github.com/microsoft/WSL/issues/7430>

----

So...  The way forward appears to be blocked for reasons as yet unknown.

Time for a "Plan B"!

SPOILER: I did end up getting the conversion to work so [click here](#hang-on-i-got-the-conversion-to-work) to cut to the chase!

----

## Create New Ubuntu 20 Distribution Under WSL 2

- <https://www.stevefenton.co.uk/2022/02/working-with-wsl-distributions/>

This looks like a promising approach: export my existing Ubuntu WSL 1 distro and import it as a new, separate Ubuntu distro under WSL 2.  

However, this was (also) scuppered by the basic fact that I can't get WSL 2 to work on my workstation.

```powershell
PS D:\> wsl --import UbuntuV2 D:\WSLDistros\ D:\WSLExports\sk_ubuntu_20.tar

# This is the same error / unsuccessful outcome as seen previously.
Please enable the Virtual Machine Platform Windows feature and ensure virtualization is enabled in the BIOS.
For information please visit https://aka.ms/wsl2-install
```

### Plan B Sources

- <https://superuser.com/questions/1515246/how-to-add-second-wsl2-ubuntu-distro-fresh-install>
- <https://www.reddit.com/r/bashonubuntuonwindows/comments/d7hst1/wsl_2_questions_multi_same_distro_boot_start/>
- <https://julienharbulot.com/wsl-dev-environment-2020.html>

----

## Install WSL 2 From Scratch

It may have to come to this, but I don't really want to lose my existing Ubuntu distribution.

- <https://www.omgubuntu.co.uk/how-to-install-wsl2-on-windows-10>

----

## Hang On!  I Got the Conversion to Work

The following article helped me to (eventually) see the light!

- <https://www.configserverfirewall.com/windows-10/please-enable-the-virtual-machine-platform-windows-feature-and-ensure-virtualization-is-enabled-in-the-bios/>

My suspicion was right: the missing piece of the puzzle wasn't in the Windows OS, it was in the properties of the VMware-hosted VM.

![Enable nested virtualisation](VMware-VM-Enable-Virtualisation.jpg)

After this setting was enabled, I tried the WSL 1 to WSL 2 conversion process again and it worked.  Hooray!

```powershell
PS D:\> wsl --status
Default Distribution: Ubuntu-20.04
Default Version: 2

Windows Subsystem for Linux was last updated on 23/06/2022
WSL automatic updates are on.

Kernel version: 5.10.102.1

PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         1

PS D:\> wsl --set-version Ubuntu-20.04 2
Conversion in progress, this may take a few minutes...
For information on key differences with WSL 2 please visit https://aka.ms/wsl2
Conversion complete.

PS D:\> wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         2

# Start the distribution.
PS D:\> wsl --distribution Ubuntu-20.04

Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.10.102.1-microsoft-standard-WSL2 x86_64)

# Success!
```
