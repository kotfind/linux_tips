# Creating bootable liveUSB 

```
dd bs=4M status=progrss oflag=sync if=archlinx.iso of=/dev/sdx
```

# After booting from liveUSB

## Check if UEFI is enabled

```
ls /sys/firmware/efi/efivars
```

The directory should exist.

## Disk partition

`fdisk`, GPT table

| Device    | Size | Type |
| :-------- | ---: | :---- |
| /dev/sda1 | 100M | EFI System (1) |
| /dev/sda2 |  16G | Linux swap (19)|
| /dev/sda3 |  ... | Linux filesystem (20) |

## Create filesystem

```
mkfs.vfat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

## Reflector

```
pacman -Syy
pacman -S reflector
reflector -c RU -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
```

## Install linux

```
mount /dev/sda3 /mnt
pacstrap /mnt base linux linux-firmware gvim
```

## Configuration

### Fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

```
arch-chroot /mnt
```

### Timedatectl

```
timedatectl list-timezones
timedatectl set-timezone Europe/Moscow
timedatectl set-ntp 1
```

### Locale

```
echo -e "en_US.UTF-8 UTF-8\nru_RU.UTF-8 UTF-8" > /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Network

```
echo kotfindPC > /etc/hostname

vim /etc/hosts # Write following:
# 127.0.0.1   localhost
# ::1         localhost
# 127.0.0.1   kotfindPC
```

### Root Password

```
passwd
```

## Grub

```
pacman -S grub efibootmgr

mkdir /boot/efi
mount /dev/sda1 /boot/efi

grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```


# Additional configuration

## Man
```
sudo pacman -S man
```

## PulseAudio

```
sudo pacman -S pulseaudio pavucontrol
```

## NetworkManager

```
pacman -S networkmanager
systemctl enable NetworkManager.service
```

## Users

```
pacman -S sudo
EDITOR=vim visudo # Uncomment `%wheel ALL=(ALL:ALL) ALL`

useradd -mG users,wheel,audio,video,input kotfind
passwd kotfind
```

## AUR (with yay)

```
pacman -S base-devel git
sudo -u kotfind -i
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
exit
```

## Haskell (with Cabal)

```
pacman -S ghc cabal-install
cabal update

vim ~/.cabal/config # Uncomment following:
# library-vanilla: False
# shared: True
# executable-dynamic: True
# program-default-options
#   ghc-options: -dynamic
```

`cabal v2-install --ghc-options=-dynamic package` to install package.

## SSH

```
sudo pacman -S openssh
ssh-keygen -t rsa
```

### Dotfiles

```
git clone git@github.com/kotfind/dotfiles.git
cd dotfiles
./install.sh
```

### Pass

```
sudo pacman -S pass xclip
git clone git@github.com/kotfind/pass.git ~/.password-store
gpg --import public.gpg
gpg --import secret.gpg
echo "pinentry-program /usr/bin/pinentry-tty" > ~/.gnupg/gpg-agent.conf
gpg-connect-agent reloadagent /bye
```

## Adding entry to fstab

```
mkdir /hdd
sudo -e /etc/fstab # Add following:
/dev/sdb1 /hdd ext4 defaults,noatime 0 3
```

## Qutebrowser

```
sudo pacman -S qutebrowser
```
