# Installing Linux

Good day. If you're not me, why are you here?

There are way better resources out there...
For instance, the [Installation guide](https://wiki.archlinux.org/title/Installation_guide) I am poorly following here.

In any case, these are the instructions and details and notes and such that I am collecting to help with my Linux setup.

## Writing the ISO file to a flash drive

Booting from an external drive makes more sense to me than the alternatives, so here we are.

First, download the ISO file from a magnet link available on [the official Arch Linux website](https://archlinux.org/download/).

Then, provided a USB drive is inserted into the current computer, it is converted into a flash drive thus.

### On MacOS

In the terminal, list the drives to get the number of the external one.
```
diskutil list
```
Then, unmount the disk, write the ISO to it and eject it.
```
diskutil unmountDisk /dev/disk2
sudo dd if=filename.iso of=/dev/rdisk2 bs=4M
diskutil eject /dev/disk2
```

### On Arch Linux

I have not yet had to perform this task, but this section will eventually contain information.

Once succefully booted into the Arch Linux ISO, there will appear a virtual console.
What to do with this is described in [the next file](ArchLinuxInstallation-ISO.md).