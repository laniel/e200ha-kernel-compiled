# Make audio work on cx2072x devices like the Asus E200HA

Most of the following information are from these sources:

- [Kernel Bug #115531](https://bugzilla.kernel.org/show_bug.cgi?id=115531)
- [Repository with the Fixes for an older kernel](https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git)
- [Asus E200HA fix-script-Repo by Grippentech](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script)

You basically need to follow the instructions in [this comment](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c64) from the bugreport. sadly they are not very detailed, so here is what needs to be done:

1. Get the Kernel. Either download it from this github site, or build it yourself (see "Building the Kernel")
2. Install the kernel by running
   ```
   dpkg -i LINUX_IMAGE_DEB_PACKAGE.deb
   dpkg -i LINUX_HEADERS_DEB_PACKAGE.deb
   ```
3. Have `alsa` and `pulseaudio` installed
4. Copy the `chtcx2072x.conf` and `HiFi.conf` (from the chtcx2072x folder) to `/usr/share/alsa/ucm/chtcx2072x`
5. Set `realtime-scheduling = no` in `/etc/pulse/daemon.conf` (see [this issue-comment](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script/issues/29#issuecomment-355113121))

# Building the kernel

## Some base info

- This Kernel ist based on the master-branch from https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git (version 4.16-rc5 at the time of writing this)
- I Applied the fixes mentioned [here](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c41) to it (See [this PR](https://github.com/heikomat/linux_with_cx2072x/pull/1))
- These fixes were for older kernels (up to 4.13), so i adjusted them to the best of my knowledge to work in the current kernel
- Keeping this up to date should be possible, though i havent tried it yet
- These instructions are written for debian

## Prerequesites

You need to have the following packages installed:
```
git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc kernel-package
```

## Configuring

- Copy a base config: `cp -v /boot/config-$(uname -r) .config`
- Run `make menuconfig`
- Enable these configurations:
  - ```
    Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> CODEC drivers -> Conexant CX2072 CODEC
    ```
  - ```
     Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> Intel ASoC SST drivers -> Intel Machine drivers > Baytrail and Cherrytrail with CX2072X codec
     ```

## Building

replace the `8` with the number of cores in your system and run:
```
fakeroot make-kpkg --initrd kernel_image kernel_headers -j 8
```

