==================================
archie - my Archlinux installation
==================================

.. image:: https://img.shields.io/github/license/Shivelight/archie.svg
   :alt: License

This document is written solely for me because I keep forgetting things. Always judge and use at your own risk. Nevertheless I accept suggestion.

Overview
========

I prefer my Archlinux to be lightweight with a small amount of eye candy. Some overview of what packages I will use for my installation:


:Bootloader:
	rEFInd_

:Display:
	NVIDIA_

:DE:
	XFCE_

:WM:
	XFWM_

:DM:
	LightDM_ with slick-greeter_

:Compositor:
	XFWM_ / Compton_

:AUR Helper:
	yay_

:Shell:
	Zsh_ with `Oh-My-Zsh <https://github.com/robbyrussell/oh-my-zsh>`_

Installation
============

Base Installation
-----------------

Follow the ArchWiki `Installation guide <https://wiki.archlinux.org/index.php/Installation_guide>`_ but do not reboot and make sure you are still in the chroot environment.

Bootloader
----------

Install rEFInd_::

	pacman -S refind-efi

Configuring Stanzas
~~~~~~~~~~~~~~~~~~~

Example stanza to put into *esp*/EFI/refind/refind.conf. Replace :code:`XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX` with your root partition PARTUUID.

::
	
	...

	menuentry "Arch Linux" {
		icon     /EFI/refind/icons/os_arch.png
		volume   "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
		loader   /boot/vmlinuz-linux
		initrd   "/boot/initramfs-linux.img"
		options  "root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw add_efi_memmap"
		submenuentry "Boot using fallback initramfs" {
			initrd "/boot/initramfs-linux-fallback.img"
		}
		submenuentry "Boot to terminal" {
			add_options "systemd.unit=multi-user.target"
		}
	}

	menuentry "Windows 10" {
		icon /EFI/refind/theme/icons/os_win8.png
		loader /EFI/Microsoft/Boot/bootmgfw.efi
	}


**TIP:** Do :code:`blkid` to see your root partition PARTUUID.

Configuring Boot Manager
~~~~~~~~~~~~~~~~~~~~~~~~

Head over to ArchWiki `UEFI#efibootmgr <https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface#efibootmgr>`_.

Post Installation
-----------------

Before rebooting, you may want to install wifi-menu dependencies to make your life easier::

	pacman -S dialog wpa_supplicant
	exit
	umount -a
	reboot

Creating User Account
~~~~~~~~~~~~~~~~~~~~~

Login with root and do::

	useradd -m -g users -G wheel -s /bin/bash <your_username>
	passwd <your_username>
	EDITOR=nano visudo

Scroll down and uncomment :code:`# %wheel ALL=(ALL) ALL`. Exit and save.

:code:`exit` and login with your user account.


AUR Helper
~~~~~~~~~~

I choose yay_ as my AUR helper, feel free to choose yours::

	sudo pacman -S git
	git clone https://aur.archlinux.org/yay.git
	cd yay
	makepkg -si

Desktop Environtment
~~~~~~~~~~~~~~~~~~~~

Upgrade your system and install XFCE_::

	sudo pacman -Syy
	sudo pacman -S xfce4

Display Manager
~~~~~~~~~~~~~~~

Install LightDM_ and slick-greeter_::

	sudo pacman -S lightdm
	yay lightdm-slick-greeter
	sudo systemctl enable lightdm.service
	sudo systemctl set-default graphical.target

Set slick-greeter as default greeter in :code:`/etc/lightdm/lightdm.conf`::

	[Seat:*]
	...
	greeter-session=lightdm-slick-greeter

Display Driver
~~~~~~~~~~~~~~

I'm using NVIDIA_ GTX 1050 Mobile card, you can refer to ArchWiki for AMD_, Intel_, or NVIDIA_ old model.

::

	sudo pacman -S nvidia

To avoid any problem, please read these:

- `NVIDIA#DRM_kernel_mode_setting <https://wiki.archlinux.org/index.php/NVIDIA#DRM_kernel_mode_setting>`_
- `NVIDIA#Pacman_Hook <https://wiki.archlinux.org/index.php/NVIDIA#Pacman_hook>`_
- `NVIDIA_Optimus#LightDM <https://wiki.archlinux.org/index.php/NVIDIA_Optimus#LightDM>`_

To fix screen tearing issue, create a new file :code:`/etc/modprobe.d/nvidia-drm.conf`::

	options nvidia_drm modeset=1

Multimedia
~~~~~~~~~~

:Sound Server:
	PulseAudio_

:Audio Mixer:
	pavucontrol_

:Framework:
	GStreamer_ (`what and why? <what_is_gst_>`_, `what?? <should_i_install_gst_>`_, `bad and ugly? <gst_bad_ugly_>`_)

:Screenshooter:
	xfce4-screenshooter_ (`alt <https://wiki.archlinux.org/index.php/Screen_capture#Screenshot_software>`_)

:Image Viewer:
	ristretto_ (`alt <https://wiki.archlinux.org/index.php/List_of_applications/Multimedia#Image_viewers>`_)

:Raster Graphic:
	GIMP_, Pinta_ (`alt and more <https://wiki.archlinux.org/index.php/List_of_applications/Multimedia#Raster_graphics_editors>`_)

:Video Player:
	SMPlayer_ (`alt <https://wiki.archlinux.org/index.php/List_of_applications/Multimedia#Video_players>`_)

:Audio Player:
	I use SMPlayer as my audio player. You can choose yours `here <https://wiki.archlinux.org/index.php/List_of_applications/Multimedia#Audio_players>`_.

.. _what_is_gst: https://fluendo.com/en/what-is-gstreamer/
.. _should_i_install_gst: https://askubuntu.com/a/926842
.. _gst_bad_ugly: https://askubuntu.com/a/468877

::

	# Sound Server
	sudo pacman -S pulseaudio pulseaudio-alsa alsa-utils alsa-plugins alsa-firmware xfce4-pulseaudio-plugin

	# Audio Mixer
	sudo pacman -S pavucontrol

	# Framework
	sudo pacman -S gstreamer
	sudo pacman -S gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly

	# Screenshooter
	sudo pacman -S xfce4-screenshooter

	# Image Viewer
	sudo pacman -S ristretto

	# Raster Graphic
	sudo pacman -S gimp pinta

	# Video Player
	sudo pacman -S smplayer

Bluetooth
~~~~~~~~~

I prefer Blueman_ as my bluetooth manager.

::

	sudo pacman -S blueman
	sudo systemctl enable bluetooth.service
	sudo systemctl start bluetooth.service

To be able to use bluetooth headphones or speakers::

	sudo pacman -S pulseaudio-bluetooth

Printers
~~~~~~~~

CUPS_ is the standars and Gutenprint_ provides drivers for Canon, Epson, Lexmark, Sony, Olympus, and PCL printers.

::

	sudo pacman -S cups cups-pdf ghostscript gsfonts gutenprint foomatic-db-gutenprint-ppds
	sudo systemctl enable org.cups.cupsd.service

If you can't print with Gutenprint_ drivers, try foomatic_::

	sudo pacman -S foomatic-db-engine foomatic-db foomatic-db-ppds foomatic-db-nonfree-ppds

Still can't print? See `CUPS/Printer-specific problems <https://wiki.archlinux.org/index.php/CUPS/Printer-specific_problems>`_.
	
Laptop
~~~~~~

See TLP_ and the `documentation <https://linrunner.de/en/tlp/docs/tlp-linux-advanced-power-management.html#installation>`_. Skip if you are not using laptop.

::

	sudo pacman -S tlp
	sudo systemctl enable tlp.service
	sudo systemctl enable tlp-sleep.service


Shell
~~~~~

Installing Zsh_ and `Oh-My-Zsh <https://github.com/robbyrussell/oh-my-zsh>`_::

	sudo pacman -S zsh
	chsh -s $(which zsh)
	zsh
	sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"


To Be Continued..
-----------------

License
=======

.. image:: https://i.creativecommons.org/l/by/4.0/88x31.png
	:alt: CC-BY-4.0
	:align: center

This work is licensed under a `Creative Commons Attribution 4.0 International License <cc-by-4.0_>`_.

.. _cc-by-4.0: https://creativecommons.org/licenses/by/4.0

.. _rEFInd: https://wiki.archlinux.org/index.php/REFInd
.. _NVIDIA: https://wiki.archlinux.org/index.php/NVIDIA
.. _XFCE: https://wiki.archlinux.org/index.php/Xfce
.. _XFWM: https://wiki.archlinux.org/index.php/Xfwm
.. _LightDM: https://wiki.archlinux.org/index.php/LightDM
.. _Compton: https://wiki.archlinux.org/index.php/Compton
.. _yay: https://github.com/Jguer/yay
.. _Zsh: https://wiki.archlinux.org/index.php/Zsh
.. _slick-greeter: https://github.com/linuxmint/slick-greeter
.. _AMD: https://wiki.archlinux.org/index.php/Xorg#AMD
.. _Intel: https://wiki.archlinux.org/index.php/Intel
.. _PulseAudio: https://wiki.archlinux.org/index.php/PulseAudio
.. _pavucontrol: https://freedesktop.org/software/pulseaudio/pavucontrol
.. _GStreamer: https://wiki.archlinux.org/index.php/GStreamer
.. _xfce4-screenshooter: https://goodies.xfce.org/projects/applications/xfce4-screenshooter
.. _ristretto: https://docs.xfce.org/apps/ristretto/start
.. _GIMP: http://www.gimp.org
.. _Pinta: http://pinta-project.com
.. _SMPlayer: https://www.smplayer.info
.. _Blueman: https://wiki.archlinux.org/index.php/Blueman
.. _CUPS: https://wiki.archlinux.org/index.php/CUPS
.. _Gutenprint: http://gimp-print.sourceforge.net
.. _foomatic: https://wiki.linuxfoundation.org/openprinting/database/foomatic
.. _Samba: https://wiki.archlinux.org/index.php/Samba
.. _TLP: https://wiki.archlinux.org/index.php/TLP