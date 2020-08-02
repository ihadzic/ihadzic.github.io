---
-title: Installing Ubuntu Linux on ASUS ROG GL12CX
-published: true
---

I recently got my hands on a new shiny ASUS ROG GL12CX desktop that
I wanted to use for robotics simulations. Yeah, what's a better platform for
robot simulations, but a gaming machine. My primary simulation
environment is Ubuntu 18.04 (Bionic) with ROS Melodic and Gazebo 9,
which means that I needed to bring up Bionic on this machine. Although
I consider myself a person who very much knows what he is doing when
it comes to Linux, I spilled quite a few pints of blood on making this
machine work that I figured it would be worth sharing some information in
public, just in case that there is someone out there trying to figure it out.

So here we go. Below are rough steps I had to go through to make it work.
As stated, my target is Ubuntu 18.04 (Bionic), but during the arduous process
I went through to figure out what was going on, I did experiment with 16.04
and 20.04 observing the same issues. So I believe that what I say below
applies to other distributions, but I have not personally verified that.

## First boot and BIOS upgrade

This machine comes with Windows 10 (what else would one expect on a
gaming PC). Before starting anything, you want to make sure that the
machine is working, so it's a good idea to just boot up the machine
as is and make sure it's sane. When you do that, you will have to go
through Windows setup process.

For me, the machine came with loosely attached SSD unit and I noticed
that before booting for the first time, because I heard something
suspiciously rattling inside the case (doesn't look good on ASUS quality
control, but whatever).

The next step is to get an Ubuntu bootable flash stick (in my case Ubuntu
18.04 Desktop). There are many great instructions how to do that and
many tools that work on both Linux and Windows, so I will not spend much
time on that here. I am an old-school guy, so I just use `dd` on another
Linux machine:

```
dd if=<iso_file> of=/dev/sd<whatever> bs=4M
```

Trying to boot up the machine from the Linux USB stick will likely
not work (it will hang at the loader) and this is the first problem that
you need to fix on this machine. The problem is in a BIOS bug. In my
case I had to upgrade the BIOS to version 310, which was released
on July 8, 2020.

I did it using the Windows utility which you can download from
[ASUS support pages](https://www.asus.com/us/Tower-PCs/ROG-Strix-GL12/HelpDesk_Download/).
Pick your machine model and download the update
utility in "BIOS Update (Windows)" section. Run it and follow the instructions.

The machine will reboot, maybe multiple times, and there will
be a short pause in BIOS during one of the reboots to actually burn the
flash. Just let it do its thing and do not power it off until it
fully comes back into Windows.

Reboot once again and press the `<F2>` key before it starts loading
Windows to enter BIOS setup. Verify that you are indeed running
version 310.

## BIOS Settings

Although Linux works with UEFI secure boot enabled, I still prefer to
turn it off if for no other reason, then to keep one variable out
of the loop. There is an option to do that in advanced Linux settings.
If you prefer to keep it on, things should still work, but I have not
verified.

The next settings you have to change to make it possible to install
Linux. This machine comes with Intel RST technology enabled. To do
that, go back to advanced BIOS settings and change the SATA mode to
AHCI. On this machine, Linux will not be able to see the SSD drive
until you do this.

If you don't plan to keep Windows, this is all you need to do, change
the mode from RST to AHCI. However, keep in mind that the moment
you do that, your Windows partition will become unbootable, so when you
start installing Linux, you will want to wipe out the whole disk/

If you want to make the machine dual-boot, you will need to carefully
follow the [instructions in Ubuntu Discourse pages](https://discourse.ubuntu.com/t/ubuntu-installation-on-computers-with-intel-r-rst-enabled/15347)
and make sure that your Windows installation still works with system
setup to use AHCI. You will also want to shrink the Windows partition
to make up the space for Linux and change Windows to use UTC
BIOS time. Anyway, I have not done this part so I won't comment much.

## Installing Linux

You are now ready to boot from Linux live USB stick. Power off
the machine, put in the USB stick, and power it on. As the machine
starts to boot, press `<F2>` to enter the BIOS and select the
USB device for a boot device. When the GRUB loader menu shows (you
may have to press `<SHIFT>` key to make it show, press `e` (for edit)
while the menu selector is pointing to the boot line. Look for the
kernel boot line. It will look something like this (not exactly but
you will recognize which line I am talking about):

```
linux   /boot/vmlinuz-5.4.0-42-generic root=UUID=a37597d5-eb1f-4abf-a8d6
-04bf22a44094 ro  quiet splash $vt_handoff
```

move your cursor to it and add `nomodeset` after the `quiet splash`.
Press `<CTRL>+<X>` to continue the boot. This step is necessary because
the machine has a NVidia GPU for which a proprietary driver is required.
It won't work with open-source Nouveau driver, so you have to disable
the GPU mode-set and run the installation in default VGA graphics mode.
The live system will boot, but the graphics will look small and ugly.
Don't worry about it for now. You will survive in this mode through the
installation.

After the live system boots, start the installation, set it up as
you please but make sure that you ***uncheck*** the option to download
updates during the installation. Also when it asks you during
the installation do ***not*** enable the automatic updates.
We will do the first update later and also enable automatic updates.
If you do it during the installation the system will not be bootable
and it will hang up at early boot time (as soon as it tries to unpack
the `initrd`). The problem is in Intel microcode update, which took me
the most time to identify as the cluprit.

## First Reboot and NVidia Driver Installation

When the installation finishes, reboot the system, remove the USB stick
when it tells you and again stop the boot in GRUB. This time you are booting
from installed system, but your graphics is still not set up, so you
still have to do the `nomodeset` change as described below. Making the machine
actually stop in GRUB may require pressing the right keys at the right time.
I have found that it's best to first stop it in BIOS (with `<F2>`) then select
(the only available) SSD drive as the boot device and immediately press the
`<SHIFT>` key to cause GRUB to show the boot menu. If you miss it and let
the sytem boot up without `nomodeset` option, you will
see the Ubuntu-typical purple screen and a couple of funny-looking
meaningless dots on the screen and you have to try again.

Once the system boots up (with crappy-looking graphics), install NVidia
proprietary device driver.
The easiest way to do this is to install it using `ubuntu-drivers` utility:

```
sudo ubuntu-drivers autoinstall
```

The utility will add the right repository, install the recommended drivers
and the necessary dependencies. The GPU that this machine has
is GeForce RTX 2060, which on my system resulted in installing the
driver version 440.10. If you look at the
[NVidia support page](https://www.nvidia.com/en-us/geforce/drivers/)
and search the recommended driver, you will find out that there is
a newer driver available (450.57 at the time of this writing), but
it is better to leave it to the Ubuntu to select one than to hand-install
the driver from NVidia web site. The latter would require manual
re-install of the driver each time you update the system. Leaving it
to Ubuntu to manage the driver from its repository will ensure that
the driver gets rebuilt correctly if the system is updated. Once
the driver is installed, reboot the system.

## Second Reboot and System Update

This time, you no longer need to stop the boot process nor change
mode settings. The machine should boot up normally and the screen resolution
will match your monitor. You should be running the NVidia proprietary
driver now. Keep in mind that the proprietary driver is not supported
by the Linux community, so any question you ask to Linux developers
will likely be ignored. You will have to take all your questions to
NVidia directly.

To verify that you are running the proprietary driver use `nvidia-smi` utility:

```
$ nvidia-smi
Sat Aug  1 19:43:11 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.100      Driver Version: 440.100      CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 2060    Off  | 00000000:01:00.0  On |                  N/A |
| 22%   39C    P8    16W / 160W |    425MiB /  5932MiB |      1%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      3329      G   /usr/lib/xorg/Xorg                            26MiB |
|    0      3689      G   /usr/bin/gnome-shell                          51MiB |
|    0      4122      G   /usr/lib/xorg/Xorg                           180MiB |
|    0      4253      G   /usr/bin/gnome-shell                         156MiB |
+-----------------------------------------------------------------------------+
```

Now is the time to update your system, but you first have to disable mark
the Intel microcode package to not be updated:

```
sudo apt-mark hold intel-microcode
```

This step is crucial. If you don't do that the update will pull the latest
3.2020 microcode which will cause your machine to hang up at boot time.
This was the step that took me the longest to figure out what was
going on. I narrowed the problem down to an early stage of boot
process when the loader unpacks the `initrd` and switches the control
to it. There is very little visibility into what the system is
doing at that stage, but in the end I figured it out.

I don't have the explanation why this happens, but all I know is that
3.2020 version of the microcode hoses up this machine miserably. So
as long as you have 3.2018 or 3.2019 you should be fine. To check what
you have, use `apt list`:

```
$ sudo apt list -a intel-microcode
Listing... Done
intel-microcode/bionic-updates,bionic-security 3.20200609.0ubuntu0.18.04.1 amd64 [upgradable from: 3.20180312.0~ubuntu18.04.1]
intel-microcode/bionic,now 3.20180312.0~ubuntu18.04.1 amd64 [installed,upgradable to: 3.20200609.0ubuntu0.18.04.1]
```

Notice above that it says that 3.2018 is installed and that 3.2020 is
available but because you put it on hold, it won't be installed. You
can also check that the offending package is indeed on hold:

```
$ sudo apt-mark showhold
intel-microcode
```

Now you can safely update your machine:

```
sudo apt update
sudo apt upgrade
```

The update process will tell you that one package is on hold, which is the
Intel microcode. After the update, reboot the machine and check that
everything is working. You can then enable the automatic update
if you desire so.

## Concluding Remarks

After almost giving up and returning the machine to the vendor,
I got it to work. I should say that it's a really nice machine.
Robotics simulations are like games. They simulate the physics
of the robot interacting with the environment, they animate
the world, and robotics control software does a lot of non-trivial
calculations in real time.

Gazebo has a metric called real-time factor that indicates the
performance of the simulation. It is the ratio between one second
of simulated time and the actual time it takes to simulate it.
The real-time factor of 1.0 means that the simulation is able
to keep up with real time.

Before I got this machine I was happy if I can achieve the
real-time factor between 0.6 and 0.7. Now, I have no problem
hitting 1.0 and still having plenty of compute capacity to
do other work while the simulation is running.
Nice end-result, but a lot of blood spilled over it.
