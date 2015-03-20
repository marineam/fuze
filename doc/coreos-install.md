# coreos-install design

## Overview

The new `coreos-install` replaces the previous bash implementation
maintained in the [init project](https://github.com/coreos/init/). The
old implementation had exactly responsibility: download a disk image,
write it to a local block device, and optionally add a cloud config. No
other functionality was allowed to keep the script relatively
maintainable. The new version initially will do the exact same thing but
offers the opportunity to improve error handling and extend
functionality over time.

## Requirements

There are two distinct development steps: *rewrite the old script* and
*integration with ignition*. It should complement any other ignition
front ends that we provide, `coreos-install` merely being a version that
also writes out disk images instead of just ignition config.

### Rewrite

The new implementation should replace the old script as soon as it
reaches feature parity with the old script:

 - Download a raw (.bin) disk image from our `*.release.core-os.net`
   sites or an alternate site defined by `$BASE_URL` or `-b`. The
   specific image may be selected by the `-V` (version), `-C` (channel),
   and `-o` (OEM) options.

 - Verify the image using the `buildbot@coreos.com` PGP key.

 - Decompress the image and write it to the block device specified by
   the `-d` option.

 - On error the disk should be left without a partition table, ensuring
   invalid images are not bootable.

 - If a cloud config is provided by the `-c` option it should be copied
   to `/var/lib/coreos-install/user_data` in the ROOT partition. The
   config should be validated and rejected before anything is written to
   disk.

### Ignition

Possible features that can be built on top of Ignition:

 - Interactive mode that with some basic prompts for common
   configuration options: user password, static IP configuration, or
   etcd discovery URL.

 - Run Igniton early instead of waiting for the initrd. This should
   probably be limited to CoreOS PXE and ISO systems to avoid runtime
   compatibility issues with the host system configuration (e.g.
   different network interface names). Running Igniton early should
   still be effectively the same as running it in an initrd but ensures
   any failures are reported during the initial install process.
