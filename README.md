# archlinux-gnome-installation-guide
archlinux和gnome桌面环境安装的完整流程。包含驱动、常用软件、推荐扩展、美化、虚拟机、性能优化、笔记本显卡切换

# 安装系统
## 脚本安装
* 连接wifi
```
iwctl #开启wifi连接
station wlan0 connect wifi名 
在跳出的条目内输入密码
exit退出iwctl
```
* ping一个网址确认网络连接
* archinstall开启安装脚本
* mirror不需要手动选，reflector会自动选择然后按照下载速度排序。在第三项里开启32位支持（multilib）
* 手动分盘：
  * 512MB FAT32 /boot
  * 内存*0.6GB swap
  * 剩余全分到一个btrfs类型分区，设置compress，添加subvolume 。这里的subolue必须是@对应/ 和 @home 对应/home，否则之后无法用timeshift创建btrfs快照
* bootload选择systemd-loader或者grub。systemd更轻量化，grub可以方便在后面设置快照启动项、主题美化、双系统启动项等等功能。systemd也可以设置，但是麻烦。
* 设置root
* 设置普通用户，添加管理员权限
* profile可以预装桌面环境，我选择minimal最小安装
* networ configuration选择第三项networkmanager，因为要安装gnome
* 开始安装

## 手动安装
参考链接：https://arch.icekylin.online/

## 双系统
### systemd-loader在bios内手动切换启动项
### grub启动
* 脚本安装完成后，选择Exit archinstall退出安装脚本
* lsblk查看winsows efi分区的设备名 #通常为某块硬盘的p1，大小100MB
* 挂载windows的efi分区

```
mount 分区名 /mnt/winboot
```

 * change root进入安装的arch系统

```
arch-chroot /mnt
```

* 安装grub、grub的组件和vim文本编辑器

```
pacman -S grub efibootmgr os-prober vim #efibootmgr是uefi启动需要的组件，os-prober是查找win11需要的组件
```
* 安装grub
```
grub-install --target=x86_64-efi --efi-directory=/boot
```

* 编辑grub配置文件
```
sudo vim /etc/default/grub
手动写入或者取消GRUB_DISABLE_OS_PROBER=false的注释
```
* 生成grub
```
grub-mkconfig -o /boot/grub/grub.cfg
```


---

# 配置系统
## 开启网络服务
```
sudo systemctl enable --now NetworkManager
```
## 安装桌面环境及必要组件
```
pacman -S gnome-desktop gdm ghostty gnome-control-center gnome-software flatpak
```
```
#gnome-desktop最小化安装gnome
#gdm是显示管理器(gnome display manager)
#ghostty是一个可高度自定义的终端模拟器（terminal emulator)
#gnome-control-center是设置中心
#software和flatpak是软件商城
```

## 创建用户
(archinstall安装的可以跳过)
```
useradd -m -g wheel -s <username> #不需要输入<>符号
```
* 设置密码
```
passwd <username>
```
* 编辑权限
```
EDITOT=vim visudo
```
* 搜索 wheel，取消注释
```
%wheel ALL=（ALL：ALL） ALL
```
## 安装N卡显卡驱动和硬件编解码
*参考链接https://wiki.archlinux.org/title/NVIDIA*

### 安装显卡驱动 

N卡此时如果不安装显卡驱动，可能无法启动桌面环境，此处以4060为例
```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```
非stable内核要安装的驱动不一样，具体看wiki，例如，zen内核装nvidia-dkms，lts装nvidia-lts

* 命令行使用 sudo nvidia-settings可以对显卡进行超频
#### AMD核显建议检查是否安装vulkan驱动
```
sudo pacman -S vulkan-radeon lib32-vulkan-radeon 
```
```
sudo pacman -S vulkan-tools
#测试工具，vulkaninfo --summary查看vulkan是否启用，vkcube测试渲染
```
- 混合模式软件还是跑在N卡上的话检查有没有安装vulkan-mesa-layers
https://www.reddit.com/r/gnome/comments/1irvmki/gnomeshell_uses_dgpu_instead_of_igpu/

```
sudo pacman -S vulkan-mesa-layers
```
### 硬件编解码
 - nvidia4060
```
sudo pacman -S libva-nvidia-driver libva libva-utils
```
- intel xe核显
```
sudo pacman -S intel-media-driver libva libva-utils
```
* amd 780M
确认安装了libva-mesa-driver
```
sudo pacman -Q libva-mesa-driver
```
* 使用vainfo确认是否安装完成
```
vainfo
```
* 环境变量名
```
LIBVA_DRIVER_NAME=nvidia
LIBVA_DRIVER_NAME=radeonsi
```

* 安装字体
```
sudo pacman -S wqy-zenhei noto-fonts
```
* 重启激活显卡驱动
```
reboot 
```
* 临时开启GDM
```
sudo systemctl start gdm #即使出了问题重启也能恢复，避免进不了tty的情况
```
* 设置gdm开机自启
```
sudo systemctl enable gdm
```
* 可选：开启32位源 (archinstall可以跳过)
```
sudo vim /etc/pacman.conf #编辑pacman配置文件
去掉[multilib]两行的注释
sudo pacman -Syyu #刷新源
```
## 生成home下目录（如果没有的话）
```
xdg-user-dirs-update
```
## 删除或隐藏不必要的快捷方式
- 编辑配置文件
可以编辑.desktop文件，写入NoDispaly=true可以在overview里隐藏图标
- 用软件管理
商城下载MenuLibre，想隐藏的图表选择hide from menus，然后保存
- 删除
*会导致软件图标丢失*
路径```/usr/share/applications```或者``` ~/.local/share/arrplications```
```
sudo rm -fv <文件名1> <文件名2>
```
删除avahi qvidcap qv4l2 ssh vnc vim org.gnome.extension



## 安装声音固件和声音服务

- 安装声音固件

```
sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf
```
- 安装声音服务
```
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```
* 启用服务
```
systemctl --user enable pipewire pipewire-pulse wireplumber
systemctl --user start pipewire pipewire-pulse wireplumber
```
* 可选：安装GUI
```
sudo pacman -S pavucontrol 
```
## 安装高级网络配置工具nm-connection-editor
```
sudo pacman -S network-manager-applet dnsmasq
```
* 设置跃点
```
启动安装的软件或输入nm-connection-editor
跃点需手动设置为100,默认的-999会导致网络速率异常
```
## 安装nano编辑器和firefox浏览器
```
sudo pacman -S nano firefox
```
* 启用firefox的硬件编码 
```
参考https://github.com/elFarto/nvidia-vaapi-driver/#firefox*
```
## 安装yay
- 编辑pacman配置文件
```
sudo nano /etc/pacman.conf
```
- 在文件底部写入以下内容
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch 
Server = https://repo.huaweicloud.com/archlinuxcn/$arch 
```
- 同步数据
```
sudo pacman -Syyu 
```
- 安装密钥
```
sudo pacman -S archlinuxcn-keyring 
```
- 安装yay
```
sudo pacman -S yay 
```

## 自定义安装软件
这是我会安装的，你可以按需求选择
- mission center是很好用的类似win11的任务管理器，使用命令安装或者商城内搜索
```

flatpak install flathub io.missioncenter.MissionCenter
```

```
sudo pacman -S gnome-text-editor gnome-disk-utility gnome-clocks gnome-calculator loupe snapshot baobab vlc fragments file-roller
```
```
#gnome-text-ditor记事本
#gnome-disk-utility磁盘管理
#gnome-clocks时钟
#gnome-calculator计算器
#loupe图像查看
#snapshot相机，摄像头
#baobab磁盘使用情况分析工具，
#vlc视频播放器
#fragments是符合gnome设计理念的种子下载器
#file-roller解压
```
```
yay -S linuxqq wechat wps-office-cn  
```

qq什么的也可以商城搜索安装，会快一些，或者用 flatpak命令:
```
flatpak install flathub com.qq.QQ com.tencent.WeChat com.wps.Office
```
- gnome截图工具自带的录屏，需登出
```
sudo pacman -S gst-plugin-pipewire gst-plugins-good
```
- gradia编辑截图
可以对截图进行一些简单的添加文字、马赛克、图表、背景之类的操作
安装：
```
flatpak install flathub be.alexandervanhee.gradia
```
设置快捷键命令：
```
flatpak run be.alexandervanhee.gradia
```
我设置了两个截图快捷键，ctrl+alt+a普通截图（仿qq截图快捷键），super+shift+s截图并进入编辑界面（仿win截图快捷键）。
## 安装输入法
- ibpinyin是中文拼音输入法，anthy是日文输入法登出一次，设置里找到键盘，添加输入源
```
sudo pacman -S ibus ibus-libpinyin ibus-anthy
```
### 设置系统语言
右键桌面选择setting，选择system，选择region&language

如果是archinstall安装，这里只有英文选项，解决办法：

* 本地化设置
```
sudo vim /etc/locale.gen 
```
```
取消zh_CN.UTF-8的注释
```
```
sudo locale-gen
```

### 配置输入法
 * 中文输入法
常规里勾选候选词，设置候选词排序为词频
拼音模式里启用云输入
辞典里勾选辞典
用户数据里取消所有勾选
* 登出，测试输入是否正常

## 快照
**快照相当于存档，每次动作之前最好都存个档**
- 安装timeshift
```
sudo pacman -S timeshift 
```
- 开启自动备份服务
```
sudo systemctl enable --now cronie.service 
```
### 自动生成快照启动项
- 安装必要组件
```
sudo pacman -S grub-btrfs 
```
- 开启服务
```
sudo systemctl enable --now grub-btrfsd.service 
```
- 修改配置文件
```
sudo systemctl edit grub-btrfsd.service 
```
- 在默认位置添加
```
[Service]
ExecStart=
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
```
- 重启服务
```
sudo systemctl daemon-reload
sudo systemctl restart grub-btrfsd.service
```
## open in any terminal
这是一个在文件管理器“右键在此处打开终端”的功能
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
修改配置，路径为/com/github/stunkymonkey/nautilus-open-any-terminal
```
- 重载nautilus
```
nautilus -q 
```
## 休眠
- 获取swap分区的UUID：ctrl+shift+c复制
```
lsblk -o name,mountpoint,size,uuid 
```
- 编辑grub
```
sudo nano /etc/default/grub 
```
- 在GRUB_CMDLINE_LINUX_DEFAULT里添加
```
resume=UUID=刚才复制的UUID
```

## 配置系统快捷键
右键桌面打开设置，选择键盘>查看及自定义快捷键
我的配置：

* 导航
```
alt+数字键 #将窗口移到工作区
Super+数字键 #切换工作区
alt+tab #切换应用程序
super+M #隐藏所有正常窗口
alt+` #在应用程序的窗口之间切换窗口
```
* 截图
```
ctrl+alt+A #交互式截图
```
 * 无障碍
```
全部backspace退格键禁用
```
* 窗口
```
super+Q #关闭窗口
Alt+super+F #切换全屏
super+F #切换最大化状态
```
* 系统
```
ctrl+super+S #打开快速设置菜单
super+G #显示全部应用
```
* 自定义快捷键<快捷键>   <命令>
```
super+B   firefox
super+T   ghostty
ctrl+alt+S    flatpak run io.missioncenter.MissionCenter
super+E   nautilus
```

## 功能性扩展
```
商店里下载Refine
#可以设置一些字体、主题和最小化最大化什么的
```
```
商店搜索extension，下载蓝色的extension
```
```
#安装扩展

AppIndicator and KStatusNotifierItem Support #右上角显示后台应用

caffeine #防止熄屏

clipboard indicator #剪贴板历史

GNOME Fuzzy App Search #模糊搜索

steal my focus window #如果打开窗口时窗口已经被打开则置顶

tiling shell#窗口平铺，tilingshell是用布局平铺,另一个叫forge是hyprland那种自动平铺但是很卡。推荐用tilingshell，记得自定义快捷键，我快捷键是super+w/a/s/d对应上下左右移动窗口，Super+Alt+w/a/s/d对应上下左右扩展窗口，super+c取消平铺。
#其他建议开启的设置选项
#启用自动平铺
#聚焦窗口边框（取消智能边框圆角弧度，宽度设置为2）

vitals #右上角显示当前资源使用情况
```
## 游戏
### steam
```
sudo pacman -S steam
```
### wine
```
sudo pacman -S wine
```
```
wincfg
```
### lutris
```
sudo pacman -S lutris
```
---

# 美化
## 更换壁纸
```
右键桌面选择更换背景
```
## 扩展美化
```
#安装扩展
lock screen background #更换锁屏背景
blur my shell #透明度美化
hide top bar #隐藏顶栏
burn my windows #应用开启和打开的动画
user themes #主题，浏览器搜索gnome shell theme下载主题
```
## 主题美化
商店里下载refine
### 光标主题
主题下载网站 https://www.gnome-look.org/browse?cat=107&ord=latest
将下载的.tar.gz文件里面的文件夹放到～/.local/share/icons/目录下，没有icons文件夹的话自己创建一个
### gnome主题
https://www.gnome-look.org/browse?cat=134&ord=latest
通常下载页面都有指引，文件路径是~/.themes/，放进去之后在user themes扩展的设置里面改可以改
## 终端美化
### 语法高亮和自动补全
- 安装终端字符字体
```
sudo pacman -S ttf-jetbrains-mono-nerd
```
 - 安装zsh
```
sudo pacman -S zsh
```
- 修改shell为zsh
```
chsh -s /usr/bin/zsh
```
```
#登出
```
```
#启动终端按0生成默认的配置文件
```
- 下载文件
```
curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```
- 编辑配置文件
```
vim ~/.zimrc
```
- 在文件底端写入 
```
zmodule romkatv/powerlevel10k
```
- 启动配置脚本
```
source ~/.zshrc
```
```
#按照引导进行选择或者抄我的，yyy1y2232121y1y
```
### ghostty美化
- 下载catppuccin颜色配置，粘贴到~/.config/ghostty/themes/
```
https://github.com/catppuccin/ghostty?tab=readme-ov-file
```
- 修改~/.config/ghostty/conf 配置文件，例如下载的是frappe的话：
```
theme = catppuccin-frappe.conf
```
- 隐藏标题栏
```
window-decoration = none
```
- 设置透明度
```
background-opacity=0.8
```
- 设置字体和字体大小
```
font-family = "Adwaita Mono" 
font-size = 15
```
# 其他
## 桌面端性能优化
### cpu资源优先级
```
sudo pacman -S ananicy-cpp cachyos-ananicy-rules-git
```
```
sudo systemctl enable --now ananicy-cpp.service
```
### N卡动态功耗调节 
*可以在笔记本功耗余量充足的情况下让4060 80w的功耗限制提高到95w以提高性能*
```
sudo systemctl enable --now nvidia-powerd.service
```

### 安装zen内核
* 安装显卡驱动，用nvidia-dkms替换nvidia驱动
```
sudo pacman -S nvidia-dkms
```
* 安装内核
```
sudo pacman -S linux-zen linux-zen-headers
```
* 重新生成grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
* 重启
```
reboot #重启时在grub的arch advance启动项里选择zen
```
* 确认正常运行后删除stable内核
```
sudo pacman -R linux 
```
* 重新生成grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### 安装alhp 
（下载太慢，容易下载失败，我不建议使用）
*参考链接https://www.bilibili.com/opus/745324585822453908?from=search&spm_id_from=333.337.0.0*

* 检查芯片支持,记住结果里是x86-64-v几
```
/lib/ld-linux-x86-64.so.2 --help
```
* 安装密钥和镜像列表
```
yay -S alhp-keyring alhp-mirrorlist
```
* 编辑配置文件
```
sudo vim /etc/pacman.conf
```
- 搜索core，在core上方加入
```
   [core-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [extra-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [multilib-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
```
* 刷新源
```
sudo pacman -Syyu
```

## KVM虚拟机
* 安装虚拟机核心组组件，图形界面， TPM
```
sudo pacman -S qemu-full virt-manager swtpm 
```
* 开启libvirtd系统服务
```
sudo systemctl enable --now libvirtd
```
* 开启NAT default网络
```
sudo virsh net-start default
sudo virsh net-autostart default
```
* 添加组权限 需要登出
```
sudo usermod -a -G libvirt $(whoami)
```
* 编辑配置文件提高权限
```
sudo vim /etc/libvirt/qemu.conf
```
```
#把user = "libvirt-qemu"改为user = "用户名"
#把group = "libvirt-qemu"改为group = "libvirt"
#取消这两行的注释
```
* 重启服务
```
sudo systemctl restart libvirtd
```
### 配置桥接网络
* 启动高级网络配置工具
```
nm-connection-editor
```
```
#添加虚拟网桥，接口填bridge0或者别的
```
```
#添加网桥连接，选择以太网，选择网络设备
```
```
#保存后将网络连接改为刚才创建的以太网网桥连接
```
### 3d加速
显示协议里监听类型选无，OpenGL，选择AMD显卡（N卡暂时不支持3d加速，可以用vmware），显卡里选virtio，勾选3d加速
### 安装win11 LTS虚拟机
* 下载win11 iot LTS iso 镜像
```
https://go.microsoft.com/fwlink/?linkid=2270353&clcid=0x409&culture=en-us&country=us
```
* 下载virtiowin11 iso
```
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.271-1/virtio-win-0.1.271.iso
```
```
根据视频指引安装
```
* 和本机进行文件分享
*参考链接https://zhuanlan.zhihu.com/p/645234144*
```
确认开启共享内存
```
```
打开文件管理器，复制要共享的文件夹的路径
```
```
在虚拟机管理器内添加共享文件夹,粘贴刚才复制的路径，取个名字
```
```
虚拟机内win11安装winFSP
https://winfsp.dev/rel/
```
```
搜索service（服务），启用VirtIO-FS Service，设置为自动
```
### 显卡直通
https://wiki.archlinuxcn.org/wiki/%E4%BD%BF%E7%94%A8_OVMF_%E7%9B%B4%E9%80%9A_PCI#
- 确认iommu是否开启，有输出说明开启
```
sudo dmesg | grep -e DMAR -e IOMMU
```
- 获取显卡的硬件id，如果是显卡单独一个group的话继续往下看，不是的话去看wiki
```
for d in /sys/kernel/iommu_groups/*/devices/*; do 
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```
- 隔离GPU
```
sudo vim /etc/modprobe.d/vfio.conf
```
写入如下内容（如果组里有多个的话这里都要填进去，逗号隔开）
```
options vfio-pci ids=硬件id,硬件id
```
- 让vfio-pci抢先加载
```
sudo vim /etc/mkinitcpio.conf
```
MODULES=（）里面写入```vfio_pci vfio vfio_iommu_type1 vfio_virqfd```
```
MODULES=(... vfio_pci vfio vfio_iommu_type1 vfio_virqfd ...)
```
HOOKS=()里面写入 modconf
```
HOOKS=(... modconf ...)
```
- 重新生成
```
sudo mkinitcpio -p linux #此处要是自己的内核，例如zen的话是linux-zen
```
- 重启电脑
#### 创建显卡直通虚拟机
virt-manager的虚拟机页面内添加设备，PCI Host Device里找到要直通的显卡。 然后USB hostDevice里面把鼠标键盘也直通进去。
#### 可以
## 笔本显卡切换
***！！！警告！！！做好快照存档***

### 切换为集显模式
#### asus华硕用户可以用supergfxctl
https://gitlab.com/asus-linux/supergfxctl
```
yay -S supergfxctl
```
```
sudo systemctl enable --now supergfxd
```
```
扩展下载GPU supergfxctl switch
```
```
使用方法：
Integrated supergfxctl --mode Integrated 
Hybrid supergfxctl --mode Hybrid 
VFIO supergfxctl --mode Vfio 
AsusEgpu supergfxctl --mode AsusEgpu 
AsusMuxDgpu supergfxctl --mode AsusMuxDgpu
```
#### envycontrol
* 笔记本BIOS内切换为混合模式
```
yay -S envycontrol 
```
* 安装gnome插件,GPU Profile Selector
```
https://extensions.gnome.org/extension/5009/gpu-profile-selector/
```
* 在右上角切换显卡至integrated

### 混合模式下用独显运行程序
####  PRIME
```
sudo pacman -S nvidia-prime
```
- 命令行内使用 prime-run命令使用独显运行软件
```
prime-run firefox 
```
- 使用menulibre修改.desktop文件，在command的最前面加上 prime-run 

#### 在gnome桌面环境下右键快捷方式选择使用独显运行
```
sudo pacman -S switcheroo-control 
```
```
sudo systemctl enable --now switcheroo-control 
```
## 电源管理
### power-profiles-daemon
*比tlp更加轻量化的电源管理，与gnome集成很好,但是不可自定义配置，可以在右上角切换模式*
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
设置方法参考官方文档https://linrunner.de/tlp/settings/index.html

这里给一个现代电脑的通用设置：
```
processor选项卡中

CPU DRIVER OPMODE 
AC active
BAT active

CPU SCALING GOVERNOR 
AC schedutil
BAT powersave

CPU ENERGY PERF POLICY
AC balance_performance
BAT power

CPU BOOST
AC on
BAT off

PLATFORM PROFILE
AC balanced
BAT low-power

MEM SLEEP
BAT deep
```
- 开启服务
```
sudo systemctl enable --now tlp
```

### cpupower
*主要用来查看当前的电源策略状态，
```
sudo pacman -S cpupower
```
- 查看当前cpufreq的状态
```
cpupower frequency-info
```
* 编辑配置文件，在需要临时调整时使用
```
sudo vim /etc/default/cpupower
```
```
需要省电时goernor设置为powersave
```
```
游戏时可以限制最大频率以达到关闭睿频的效果，从而降低温度和功耗
```
* 启用服务
```
sudo systemctl start cpupower
#推荐使用start或stop命令临时开启或者关闭
```
### 插件扩展
```
power tracker #显示电池充放电
auto power profile #配合powerProfilesDaemon使用，可以自动切换模式
power profile indicator # 配合powerProfilesDaemon使用，顶栏显示当前模式
```
### 推荐管理方式
**power-profiles-daemon配合cpupower**
因为power-profiles-daemon和gnome的兼容性好，默认有三种模式可以快速在性能和省电之间切换，配置方便。在此基础上搭配cpupower在需要时进行手动微调，能达到和tlp类似的效果。如果需要最强的续航，那还是要用tlp。

## 自定义安装字体
* 复制字体到该目录下（可以自由创建子目录）
```
～/local/share/fonts/
```
* 刷新字体缓存
```
fc-cache -fv
```
## smb文件共享
如果你的路由器或者别的设备开启了smb文件共享，安装gvfs-smb可以使你在nautilus访问那些文件
```
sudo pacman -S gvfs-smb
```
## pacman常用指令

* 删除包，同时删除不再被其他包需要的依赖和配置文件 #-R删除包，s删除依赖，n删除配置文件
```
sudo pacman -Rns
```
* 查询包
```
sudo pacman -Ss
```
* 列出所有已安装的包
```
sudo pacman -Qe
```
* 列出所有已安装的依赖
```
sudo pacman -Qd
```
* 清理包缓存
```
sudo pacman -Sc
```
* 列出孤立依赖包
```
sudo pacman -Qdt
```
* 清理孤立依赖包
```
sudo pacman -Rns $(pacman -Qdt)
```
---

# 附录
* linux系统下每次涉及到依赖的操作前都要快照存档
* grub卡顿是n卡的锅
## 域名解析出现暂时性错误
```
sudo vim /etc/resolv.conf
```
内容修改为
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
## windows磁盘管理工具
```
NIUBI partition Editor free edition
```

## 美化kitty
多个显示器的情况下，kitty用tiling shell扩展的自动平铺有bug，无法在当前显示器开启第一个窗口。
```
sudo pacman -S kitty
```
```
#下载配置文件 https://github.com/catppuccin/kitty
以frappe为例，下载frappe.conf，复制到~/.config/kitty/目录下，重命名为kitty.conf
```
```
#编辑配置文件
写入：
linux_display_server x11 #修复kitty奇怪的刘海
hide_window_decorations yes #隐藏顶栏，隐藏后无法调整窗口大小，建议配合tiling shell扩展使用
background_opacity 0.8 #设置背景透明度
font_family 字体名
font_size 字体大小数字
```
```
#如果波浪号在左上角，配置文件写入：
symbol_map U+007E Adwaita Mono
#强制指定notosansmono字体，也可以选择别的
```
```
#我的示例配置
hide_window_decorations yes
background_opacity 0.8
font_family Adwaita Mono
font_size 14
```
```
#重启终端
```
## 参考资料：
https://arch.icekylin.online/guide/rookie/basic-install

https://wiki.archlinuxcn.org/wiki/

https://www.bilibli.com

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

