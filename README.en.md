# Choose language: [**ENGLISH-AI-tran.**](./README.en.md) | [中文](./README.md)


# Installing the System
## Script Installation
* Connect to Wi-Fi
```
iwctl # Turn on Wi-Fi connection
station wlan0 connect wifi_name 
Enter the password in the prompted field
exit # Exit iwctl
```
* Ping a website to confirm network connection
* Run archinstall to start the installation script
* For mirrors, no need to select manually; reflector will automatically choose and sort them by download speed. Enable 32-bit support (multilib) in the third option
* Manually partition the disk:
  * 512MB FAT32 /boot
  * Memory * 0.6GB swap
  * Allocate the remaining space to a btrfs-type partition, set compress, and add subvolumes. The subvolumes here must be @ corresponding to / and @home corresponding to /home; otherwise, you cannot create btrfs snapshots with timeshift later
* Choose bootloader: systemd-loader or grub. Systemd is more lightweight, while grub makes it easier to set up snapshot boot entries, theme customization, dual-boot options, and other features later. Systemd can also be configured, but it’s more troublesome
* Set root
* Create a regular user and add administrative privileges
* Profile can pre-install a desktop environment; I chose minimal installation
* For network configuration, select the third option, NetworkManager, because we are installing GNOME
* Start the installation

## Manual Installation
Reference link: https://arch.icekylin.online/

## Dual System
### systemd-loader: Manually Switch Boot Entries in BIOS
### grub Boot
* After script installation is complete, choose Exit archinstall to exit the installation script
* Use lsblk to check the device name of the Windows EFI partition # Usually p1 of a certain disk, size 100MB
* Mount the Windows EFI partition

```
mount partition_name /mnt/winboot
```

 * Change root into the installed Arch system

```
arch-chroot /mnt
```

* Install grub, grub components, and vim text editor

```
pacman -S grub efibootmgr os-prober vim 

# efibootmgr is a component needed for UEFI boot, os-prober is a component needed to detect Windows 11
```
* Install grub
```
grub-install --target=x86_64-efi --efi-directory=/boot
```

* Edit the grub configuration file
```
sudo vim /etc/default/grub

Manually write or uncomment GRUB_DISABLE_OS_PROBER=false
```
* Generate grub configuration
```
grub-mkconfig -o /boot/grub/grub.cfg
```


---

# Configuring the System
## Enable Network Service
```
sudo systemctl enable --now NetworkManager
```
## Install Desktop Environment and Essential Components
```
pacman -S gnome-desktop gdm ghostty gnome-control-center gnome-software flatpak
```
```
# gnome-desktop: Minimal GNOME installation
# gdm: Display manager (GNOME Display Manager)
# ghostty: A highly customizable terminal emulator
# gnome-control-center: Settings center
# software and flatpak: Software store
```

## Create User
(If installed via archinstall, you can skip this)
```
useradd -m -g wheel -s <username> # No need to enter <> symbols
```
* Set password
```
passwd <username>
```
* Edit permissions
```
EDITOR=vim visudo
```
* Search for wheel, uncomment the following line
```
%wheel ALL=(ALL:ALL) ALL
```
## Install NVIDIA Graphics Driver and Hardware Decoding
*Reference link: https://wiki.archlinux.org/title/NVIDIA*

### Install Graphics Driver 

For NVIDIA cards, if the graphics driver is not installed at this point, the desktop environment may not start. Here, we use the 4060 as an example
```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```
For non-stable kernels, the drivers to install differ; check the wiki for details. For example, use nvidia-dkms for the Zen kernel, nvidia-lts for LTS

* Use the command sudo nvidia-settings to overclock the graphics card from the command line
#### For AMD Integrated Graphics, Check if Vulkan Driver is Installed
```
sudo pacman -S vulkan-radeon lib32-vulkan-radeon 
```
```
sudo pacman -S vulkan-tools
# Testing tools: vulkaninfo --summary to check if Vulkan is enabled, vkcube to test rendering
```
- If software in hybrid mode still runs on the NVIDIA card, check if vulkan-mesa-layers is installed
https://www.reddit.com/r/gnome/comments/1irvmki/gnomeshell_uses_dgpu_instead_of_igpu/

```
sudo pacman -S vulkan-mesa-layers
```
### Hardware Decoding
 - NVIDIA 4060
```
sudo pacman -S libva-nvidia-driver libva libva-utils
```
- Intel Xe Integrated Graphics
```
sudo pacman -S intel-media-driver libva libva-utils
```
* AMD 780M
Confirm that libva-mesa-driver is installed
```
sudo pacman -Q libva-mesa-driver
```
* Use vainfo to confirm installation
```
vainfo
```
* Environment variable names
```
LIBVA_DRIVER_NAME=nvidia
LIBVA_DRIVER_NAME=radeonsi
```

* Install fonts
```
sudo pacman -S wqy-zenhei noto-fonts
```
* Reboot to activate the graphics driver
```
reboot 
```
* Temporarily start GDM
```
sudo systemctl start gdm # Even if something goes wrong, rebooting will recover, avoiding inability to access tty
```
* Set GDM to start on boot
```
sudo systemctl enable gdm
```
* Optional: Enable 32-bit repository (can skip if using archinstall)
```
sudo vim /etc/pacman.conf # Edit the pacman configuration file
Uncomment the two lines under [multilib]
sudo pacman -Syyu # Refresh repositories
```
## Generate Home Directory Structure (if not present)
```
xdg-user-dirs-update
```
## Remove or Hide Unnecessary Shortcuts
- Edit configuration file
You can edit .desktop files and add NoDisplay=true to hide icons in the overview
- Use software management
Download MenuLibre from the store, select "Hide from menus" for the icons you want to hide, then save
- Delete
*This will cause software icons to disappear*
Paths: ```/usr/share/applications``` or ```~/.local/share/applications```
```
sudo rm -fv <filename1> <filename2>
```
Delete avahi, qvidcap, qv4l2, ssh, vnc, vim, org.gnome.extension



## Install Audio Firmware and Services

- Install audio firmware

```
sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf
```
- Install audio services
```
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```
* Enable services
```
systemctl --user enable pipewire pipewire-pulse wireplumber
systemctl --user start pipewire pipewire-pulse wireplumber
```
* Optional: Install GUI
```
sudo pacman -S pavucontrol 
```
## Install Advanced Network Configuration Tool nm-connection-editor
```
sudo pacman -S network-manager-applet dnsmasq
```
* Set metric
```
Launch the installed software or enter nm-connection-editor
The metric needs to be manually set to 100; the default -999 will cause abnormal network speeds
```
## Install Nano Editor and Firefox Browser
```
sudo pacman -S nano firefox
```
* Enable hardware encoding in Firefox 
```
Reference: https://github.com/elFarto/nvidia-vaapi-driver/#firefox
```
## Install yay
- Edit the pacman configuration file
```
sudo nano /etc/pacman.conf
```
- Add the following content at the bottom of the file
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch 
Server = https://repo.huaweicloud.com/archlinuxcn/$arch 
```
- Sync data
```
sudo pacman -Syyu 
```
- Install keyring
```
sudo pacman -S archlinuxcn-keyring 
```
- Install yay
```
sudo pacman -S yay 
```

## Custom Software Installation
These are what I would install; you can choose according to your needs
- Mission Center is a great task manager similar to Windows 11’s; install via command or search in the store
```

flatpak install flathub io.missioncenter.MissionCenter
```

```
sudo pacman -S gnome-text-editor gnome-disk-utility gnome-clocks gnome-calculator loupe snapshot baobab vlc fragments file-roller
```
```
# gnome-text-editor: Notepad
# gnome-disk-utility: Disk management
# gnome-clocks: Clock
# gnome-calculator: Calculator
# loupe: Image viewer
# snapshot: Camera, webcam
# baobab: Disk usage analysis tool
# vlc: Video player
# fragments: Torrent downloader following GNOME design philosophy
# file-roller: Archive manager
```
```
yay -S linuxqq wechat wps-office-cn  
```

For QQ and similar apps, you can search and install them from the store for faster installation, or use the flatpak command:
```
flatpak install flathub com.qq.QQ com.tencent.WeChat com.wps.Office
```
- GNOME’s built-in screenshot tool with screen recording; requires logout
```
sudo pacman -S gst-plugin-pipewire gst-plugins-good
```
- Gradia for editing screenshots
Allows simple operations like adding text, mosaics, shapes, backgrounds, etc. to screenshots
Install:
```
flatpak install flathub be.alexandervanhee.gradia
```
Set shortcut command:
```
flatpak run be.alexandervanhee.gradia
```
I set two screenshot shortcuts: Ctrl+Alt+A for regular screenshot (mimicking QQ’s shortcut), Super+Shift+S for screenshot and edit (mimicking Windows’ shortcut)
## Install Input Method
- ibpinyin is a Chinese pinyin input method, anthy is a Japanese input method; log out once, go to Settings, find Keyboard, and add input sources
```
sudo pacman -S ibus ibus-libpinyin ibus-anthy
```
### Set System Language
Right-click the desktop, select Settings, choose System, then Region & Language

If installed via archinstall, only English options are available here. Solution:

* Localization settings
```
sudo vim /etc/locale.gen 
```
```
Uncomment zh_CN.UTF-8
```
```
sudo locale-gen
```

### Configure Input Method
 * Chinese Input Method
In General, enable candidate words, set candidate word sorting to frequency
In Pinyin Mode, enable cloud input
In Dictionary, enable dictionary
In User Data, uncheck all options
* Log out and test if input works normally

## Snapshots
**Snapshots are like save points; it’s best to create one before any operation**
- Install timeshift
```
sudo pacman -S timeshift 
```
- Enable automatic backup service
```
sudo systemctl enable --now cronie.service 
```
### Automatically Generate Snapshot Boot Entries
- Install necessary components
```
sudo pacman -S grub-btrfs 
```
- Enable service
```
sudo systemctl enable --now grub-btrfsd.service 
```
- Modify configuration file
```
sudo systemctl edit grub-btrfsd.service 
```
- Add at the default location
```
[Service]
ExecStart=
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
```
- Restart service
```
sudo systemctl daemon-reload
sudo systemctl restart grub-btrfsd.service
```
## Open in Any Terminal
This is a feature to "right-click to open terminal here" in the file manager
```
yay -S nautilus-open-any-terminal 
```
```
sudo glib-compile-schemas /usr/share/glib-2.0/schemas 
```
```
sudo pacman -S dconf-editor
```
```
Modify configuration at the path /com/github/stunkymonkey/nautilus-open-any-terminal
```
- Reload nautilus
```
nautilus -q 
```
## Hibernation
- Get the UUID of the swap partition: Ctrl+Shift+C to copy
```
lsblk -o name,mountpoint,size,uuid 
```
- Edit grub
```
sudo nano /etc/default/grub 
```
- In GRUB_CMDLINE_LINUX_DEFAULT, add
```
resume=UUID=the_UUID_you_copied_earlier
```

## Configure System Shortcuts
Right-click the desktop, open Settings, choose Keyboard > View and Customize Shortcuts
My configuration:

* Navigation
```
Alt+number key # Move window to workspace
Super+number key # Switch workspace
Alt+Tab # Switch applications
Super+M # Hide all normal windows
Alt+` # Switch between windows of the same application
```
* Screenshots
```
Ctrl+Alt+A # Interactive screenshot
```
 * Accessibility
```
All disabled with Backspace
```
* Windows
```
Super+Q # Close window
Alt+Super+F # Toggle fullscreen
Super+F # Toggle maximized state
```
* System
```
Ctrl+Super+S # Open quick settings menu
Super+G # Show all applications
```
* Custom Shortcuts <shortcut>   <command>
```
Super+B   firefox
Super+T   ghostty
Ctrl+Alt+S    flatpak run io.missioncenter.MissionCenter
Super+E   nautilus
```

## Functional Extensions
```
Download Refine from the store
# Can set fonts, themes, minimize/maximize options, etc.
```
```
Search for "extension" in the store, download the blue "extension"
```
```
# Install extensions

AppIndicator and KStatusNotifierItem Support # Show background apps in the top-right corner

caffeine # Prevent screen timeout

clipboard indicator # Clipboard history

GNOME Fuzzy App Search # Fuzzy search

steal my focus window # Bring an already-open window to the top when reopened

tiling shell # Window tiling; tilingshell uses layout tiling, while forge is like Hyprland’s automatic tiling but laggy. Recommend tilingshell, set custom shortcuts: Super+W/A/S/D to move windows up/left/down/right, Super+Alt+W/A/S/D to expand windows up/left/down/right, Super+C to cancel tiling
# Other recommended settings
# Enable automatic tiling
# Focus window borders (disable smart border rounding, set width to 2)

vitals # Show current resource usage in the top-right corner
```
## Games
### Steam
```
sudo pacman -S steam
```
### Wine
```
sudo pacman -S wine
```
```
winecfg
```
### Lutris
```
sudo pacman -S lutris
```
---

# Beautification
## Change Wallpaper
```
Right-click the desktop and select Change Background
```
## Extension Beautification
```
# Install extensions
lock screen background # Change lock screen background
blur my shell # Transparency beautification
hide top bar # Hide top bar
burn my windows # Animation for opening and closing applications
user themes # Themes; search for "gnome shell theme" in a browser to download themes
```
## Theme Beautification
Download Refine from the store
### Cursor Theme
Theme download site: https://www.gnome-look.org/browse?cat=107&ord=latest
Extract the folder from the downloaded .tar.gz file and place it in ~/.local/share/icons/. If there’s no icons folder, create one
### GNOME Theme
https://www.gnome-look.org/browse?cat=134&ord=latest
Download pages usually have instructions; the file path is ~/.themes/. After placing it there, modify it in the settings of the user themes extension
## Terminal Beautification
### Syntax Highlighting and Autocompletion
- Install terminal font
```
sudo pacman -S ttf-jetbrains-mono-nerd
```
 - Install zsh
```
sudo pacman -S zsh
```
- Change shell to zsh
```
chsh -s /usr/bin/zsh
```
```
# Log out
```
```
# Start terminal and press 0 to generate the default configuration file
```
- Download file
```
curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```
- Edit configuration file
```
vim ~/.zimrc
```
- Add at the bottom of the file 
```
zmodule romkatv/powerlevel10k
```
- Run configuration script
```
source ~/.zshrc
```
```
# Follow the guide or copy my choices: yyy1y2232121y1y
```
### ghostty Beautification
- Download Catppuccin color scheme and paste it into ~/.config/ghostty/themes/
```
https://github.com/catppuccin/ghostty?tab=readme-ov-file
```
- Modify ~/.config/ghostty/config configuration file; for example, if you downloaded frappe:
```
theme = catppuccin-frappe.conf
```
- Hide title bar
```
window-decoration = none
```
- Set transparency
```
background-opacity=0.8
```
- Set font and font size
```
font-family = "Adwaita Mono" 
font-size = 15
```
# Others
## Desktop Performance Optimization
### CPU Resource Priority
```
sudo pacman -S ananicy-cpp cachyos-ananicy-rules-git
```
```
sudo systemctl enable --now ananicy-cpp.service
```
### NVIDIA Dynamic Power Adjustment 
*Can increase the 4060’s 80W power limit to 95W to improve performance when laptop power capacity is sufficient*
```
sudo systemctl enable --now nvidia-powerd.service
```

### Install Zen Kernel
* Install graphics driver, replace nvidia with nvidia-dkms
```
sudo pacman -S nvidia-dkms
```
* Install kernel
```
sudo pacman -S linux-zen linux-zen-headers
```
* Regenerate grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
* Reboot
```
reboot # During reboot, select Zen in grub’s Arch Advanced boot options
```
* After confirming normal operation, remove the stable kernel
```
sudo pacman -R linux 
```
* Regenerate grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Install ALHP 
(Download speed is too slow and often fails; I don’t recommend using it)
*Reference link: https://www.bilibili.com/opus/745324585822453908?from=search&spm_id_from=333.337.0.0*

* Check chip support, note the result as x86-64-vX
```
/lib/ld-linux-x86-64.so.2 --help
```
* Install keyring and mirror list
```
yay -S alhp-keyring alhp-mirrorlist
```
* Edit configuration file
```
sudo vim /etc/pacman.conf
```
- Search for core, add above core
```
   [core-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [extra-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [multilib-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
```
* Refresh repositories
```
sudo pacman -Syyu
```

## KVM Virtual Machine
* Install virtual machine core components, graphical interface, TPM
```
sudo pacman -S qemu-full virt-manager swtpm 
```
* Enable libvirtd system service
```
sudo systemctl enable --now libvirtd
```
* Enable NAT default network
```
sudo virsh net-start default
sudo virsh net-autostart default
```
* Add group permissions; requires logout
```
sudo usermod -a -G libvirt $(whoami)
```
* Edit configuration file to increase permissions
```
sudo vim /etc/libvirt/qemu.conf
```
```
# Change user = "libvirt-qemu" to user = "username"
# Change group = "libvirt-qemu" to group = "libvirt"
# Uncomment these two lines
```
* Restart service
```
sudo systemctl restart libvirtd
```
### Configure Bridged Network
* Launch advanced network configuration tool
```
nm-connection-editor
```
```
# Add a virtual bridge, set the interface to bridge0 or another name
```
```
# Add a bridge connection, select Ethernet, choose the network device
```
```
# Save and switch the network connection to the newly created Ethernet bridge connection
```
### 3D Acceleration
In Display Protocol, set Listen Type to None, OpenGL, select AMD graphics card (NVIDIA does not currently support 3D acceleration; use VMware instead), in Graphics, select virtio, and enable 3D acceleration
### Install Windows 11 LTS Virtual Machine
* Download Windows 11 IoT LTS ISO image
```
https://go.microsoft.com/fwlink/?linkid=2270353&clcid=0x409&culture=en-us&country=us
```
* Download virtio-win11 ISO
```
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.271-1/virtio-win-0.1.271.iso
```
```
Follow video instructions for installation
```
* Share files with the host
*Reference link: https://zhuanlan.zhihu.com/p/645234144*
```
Confirm shared memory is enabled
```
```
Open File Explorer, copy the path of the folder to share
```
```
In the virtual machine manager, add a shared folder, paste the copied path, and give it a name
```
```
Install WinFSP in the Windows 11 virtual machine
https://winfsp.dev/rel/
```
```
Search for Services, enable VirtIO-FS Service, set it to Automatic
```
### Graphics Card Passthrough
https://wiki.archlinuxcn.org/wiki/%E4%BD%BF%E7%94%A8_OVMF_%E7%9B%B4%E9%80%9A_PCI#
- Confirm if IOMMU is enabled; output indicates it’s enabled
```
sudo dmesg | grep -e DMAR -e IOMMU
```
- Get the hardware ID of the graphics card; if it’s in its own group, proceed; otherwise, refer to the wiki
```
for d in /sys/kernel/iommu_groups/*/devices/*; do 
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```
- Isolate GPU
```
sudo vim /etc/modprobe.d/vfio.conf
```
Add the following content (if the group has multiple IDs, list them all, separated by commas)
```
options vfio-pci ids=hardware_id,hardware_id
```
- Ensure vfio-pci loads first
```
sudo vim /etc/mkinitcpio.conf
```
In MODULES=(), add ```vfio_pci vfio vfio_iommu_type1 vfio_virqfd```
```
MODULES=(... vfio_pci vfio vfio_iommu_type1 vfio_virqfd ...)
```
In HOOKS=(), add modconf
```
HOOKS=(... modconf ...)
```
- Regenerate
```
sudo mkinitcpio -p linux # Use your kernel here, e.g., linux-zen for Zen
```
- Reboot
#### Create Graphics Card Passthrough Virtual Machine

- install ovmf
```
sudo pacman -S edk2-ovmf
```
edit conf
```
sudo vim /etc/libvirt/qemu.conf
```
```
nvram = [
	"/usr/share/ovmf/x64/OVMF_CODE.fd:/usr/share/ovmf/x64/OVMF_VARS.fd"
]
```
restart service
```
sudo systemctl restart libvirtd
```
In virt-manager’s virtual machine page, add a device; find the graphics card to pass through in PCI Host Device. Then, in USB Host Device, pass through the mouse and keyboard as well

## Laptop Graphics Switching
***!!! Warning !!! Make sure to create a snapshot archive***

### Switch to Integrated Graphics Mode
#### ASUS Users Can Use supergfxctl
https://gitlab.com/asus-linux/supergfxctl
```
yay -S supergfxctl
```
```
sudo systemctl enable --now supergfxd
```
```
Install the extension GPU supergfxctl switch
```
```
Usage:
Integrated: supergfxctl --mode Integrated 
Hybrid: supergfxctl --mode Hybrid 
VFIO: supergfxctl --mode Vfio 
AsusEgpu: supergfxctl --mode AsusEgpu 
AsusMuxDgpu: supergfxctl --mode AsusMuxDgpu
```
#### envycontrol
* Switch the laptop BIOS to hybrid mode
```
yay -S envycontrol 
```
* Install GNOME plugin, GPU Profile Selector
```
https://extensions.gnome.org/extension/5009/gpu-profile-selector/
```
* Switch graphics to integrated in the top-right corner

### Run Programs with Discrete Graphics in Hybrid Mode
#### PRIME
```
sudo pacman -S nvidia-prime
```
- Use the prime-run command in the terminal to run software with discrete graphics
```
prime-run firefox 
```
- Use MenuLibre to modify .desktop files, adding prime-run at the beginning of the command

#### Right-Click Shortcut to Run with Discrete Graphics in GNOME Desktop Environment
```
sudo pacman -S switcheroo-control 
```
```
sudo systemctl enable --now switcheroo-control 
```
## Power Management
### power-profiles-daemon
*More lightweight than TLP, integrates well with GNOME, but lacks custom configuration; switch modes in the top-right corner*
```
sudo pacman -S power-profiles-daemon
```
```
sudo systemctl enable --now power-profiles-daemon 
```

### TLP 
```
sudo pacman -S tlp tlp-rdw 
```
```
yay -S tlpui
```
Refer to the official documentation for settings: https://linrunner.de/tlp/settings/index.html

Here’s a general setup for modern computers:
```
Processor tab:

CPU DRIVER OPMODE 
AC: active
BAT: active

CPU SCALING GOVERNOR 
AC: schedutil
BAT: powersave

CPU ENERGY PERF POLICY
AC: balance_performance
BAT: power

CPU BOOST
AC: on
BAT: off

PLATFORM PROFILE
AC: balanced
BAT: low-power

MEM SLEEP
BAT: deep
```
- Enable service
```
sudo systemctl enable --now tlp
```

### cpupower
*Mainly used to check the current power policy status*
```
sudo pacman -S cpupower
```
- Check current cpufreq status
```
cpupower frequency-info
```
* Edit configuration file for temporary adjustments
```
sudo vim /etc/default/cpupower
```
```
For power saving, set governor to powersave
```
```
For gaming, limit max frequency to disable turbo boost, reducing temperature and power consumption
```
* Enable service
```
sudo systemctl start cpupower
# Recommend using start or stop commands to enable or disable temporarily
```
### Plugin Extensions
```
power tracker # Show battery charge/discharge
auto power profile # Works with powerProfilesDaemon to switch modes automatically
power profile indicator # Works with powerProfilesDaemon, shows current mode in the top bar
```
### Recommended Management Approach
**power-profiles-daemon with cpupower**
Because power-profiles-daemon integrates well with GNOME and offers three modes for quick switching between performance and power saving with easy configuration. Paired with cpupower for manual fine-tuning when needed, it achieves effects similar to TLP. For maximum battery life, use TLP.

## Custom Font Installation
* Copy fonts to this directory (you can create subdirectories freely)
```
~/local/share/fonts/
```
* Refresh font cache
```
fc-cache -fv
```
## SMB File Sharing
If your router or another device has SMB file sharing enabled, installing gvfs-smb allows you to access those files in Nautilus
```
sudo pacman -S gvfs-smb
```
## Common pacman Commands

* Remove a package, along with dependencies and configuration files no longer needed by other packages # -R removes package, -s removes dependencies, -n removes configs
```
sudo pacman -Rns
```
* Search for a package
```
sudo pacman -Ss
```
* List all installed packages
```
sudo pacman -Qe
```
* List all installed dependencies
```
sudo pacman -Qd
```
* Clean package cache
```
sudo pacman -Sc
```
* List orphaned dependency packages
```
sudo pacman -Qdt
```
* Clean orphaned dependency packages
```
sudo pacman -Rns $(pacman -Qdt)
```
---

# Appendix
* In Linux systems, it’s recommended to create a snapshot archive before performing operations involving dependencies
* Grub stuttering is due to NVIDIA cards
## Temporary Error in Domain Name Resolution
```
sudo vim /etc/resolv.conf
```
Change the content to
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
## Windows Disk Management Tool
```
NIUBI Partition Editor Free Edition
```

## Beautify kitty
When using multiple monitors, kitty has a bug with the tiling shell extension’s automatic tiling, failing to open the first window on the current monitor
```
sudo pacman -S kitty
```
```
# Download configuration file: https://github.com/catppuccin/kitty
For example, with frappe, download frappe.conf, copy it to ~/.config/kitty/, and rename it to kitty.conf
```
```
# Edit configuration file
Add:
linux_display_server x11 # Fix kitty’s strange notch
hide_window_decorations yes # Hide top bar; after hiding, window size cannot be adjusted, recommended to use with tiling shell extension
background_opacity 0.8 # Set background transparency
font_family font_name
font_size font_size_number
```
```
# If the tilde is in the top-left corner, add to the configuration file:
symbol_map U+007E Adwaita Mono
# Force specify noto-sans-mono font, or choose another
```
```
# My example configuration
hide_window_decorations yes
background_opacity 0.8
font_family Adwaita Mono
font_size 14
```
```
# Restart terminal
```
## References:
https://arch.icekylin.online/guide/rookie/basic-install

https://wiki.archlinuxcn.org/wiki/

https://www.bilibili.com

https://cn.linux-terminal.com/?p=4593

https://www.microsoft.com/zh-cn/privacy/privacystatement

https://github.com/Stunkymonkey/nautilus-open-any-terminal

https://aur.archlinux.org/packages/nautilus-open-any-terminal

https://phoenixnap.com/kb/ubuntu-install-kvm

https://missioncenter.io/

https://www.reddit.com/r/ChromeOSFlex/comments/ucno4b/qemukvm_virtmanager_windows_vm_very_slow/

https://cn.linux-terminal.com/?p=4593

https://zhuanlan.zhihu.com/p/645234144

https://github.com/bayasdev/envycontrol

https://github.com/LorenzoMorelli/GPU_profile_selector 
https://www.bilibili.com/video/BV1ym4y1G76s/?share_source=copy_web&vd_source=1c6a132d86487c8c4a29c7ff5cd8ac50

https://www.youtube.com/watch?v=8WkcLwXCFJQ&t=1399s

https://grok.com/

https://www.youtube.com/watch?v=AE1-W2bMVEs&t=316s

https://www.reddit.com/r/linux_gaming/comments/17h1i7n/linux_vs_windows_tested_in_10_games_linux_17/

https://linuxblog.io/boost-battery-life-on-linux-laptop-tlp/

https://linrunner.de/tlp/settings/index.html
