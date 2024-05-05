# goaround

make use of takeover.sh [1] for filesystem maintenance or replacing current OS installation with another distribution

CURRENTLY just a PROOF OF CONCEPT
  * don't use it on production systems !!
  * OS replacement could easily make your remote VM unbootable or worse (lots of work through rescue VNC consoles if available at all, DATA LOSS or extra costs for remote hands, a.s.o.)

currently ONLY supports:
- ubuntu 22.04 as starting OS
- fsck as maintenance script (reboot into same OS again afterwards)
- devuan [2] daedalus as destination OS for OS replacement and only when filesystem resides on a specific gpt partition layout

why goaround?
* I needed a way to install devuan on a datacenter VM where no devuan images are available for click-n-install
* i did NOT want to boot into a rescue system only to manually fiddle with debootstrap maybe even through a VNC sesson instead of ssh
* also rebooting some remote hardware boxes sometimes let them stay stuck in bios not "detecting" the harddrive any more until the next power cycle. using goaround seems to solve this issue as bios reboot is then not performed, only a kexec call which handles hardware initialization different.
* go-around from aviation seems a reasonable analogy for a script that restarts kernels without doing full bios reboots while also performing critical actions at the same time ( like changing the airplane underneath the pilots seat xD )

how does it work?
* takeover.sh[1]
  * allows full hdd and filesystem access without the running OS interfering (any more) as "/" can be fully unmounted
  * runs in a tmpfs environment and uses "fakeinit" to replace init process
  * full "fsck -yvf" on former root "/" filesystem is possible
  * OS main folders /sbin /root /usr /lib a.s.o. can be freely moved around, so that the new OS can be put in place.
* goaround.sh
  * adds binaries (like vgchange, kexec and fsck) into /takeover tmpfs mount for use within the scripts
  * provides a script for fsck'ing the root partition
  * provides a script for saving the output logfiles from tmpfs mount to disk
  * provides devuan preparation scripts around debootstrap and does the actual move of the OS folders
  * kexec -reboots into the current or the new installed kernel depending on parameters
  
current workflow for OS replacement:
1. copy scripts/files to machine
2. run prepare_devuan.sh
3. run goaround.sh wich in turn:
  - move current files and folders in "/" into /.move_folders/old/ and then moves /.move_folders/new/* to "/"
  - copies script output logfiles to disk under (the new) /var/log/goaround/
  - reboots again into the new kernel/initramfs with its parameters but using kexec
4. manually login to new OS (and if needed rerun update-grub and grub-install) - if that fails, it should still possible to manually move the old folders back into place (using a rescue system) and then reboot into the old system
5. try a "normal" reboot through "bios" to ensure bootloader works as expected
6. remove old OS remains from /.move_folders/old or migrated user data if needed.

[1] https://github.com/marcan/takeover.sh
[2] https://www.devuan.org/
