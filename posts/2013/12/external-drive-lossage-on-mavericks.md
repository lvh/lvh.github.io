<!--
.. title: External drive lossage on Mavericks
.. date: 2013/12/18 11:22
.. slug: external-drive-lossage-on-mavericks
.. link:
.. description:
.. tags: 
-->


Welp, my WD 1TB Passport drive borked. Seems to be a [common issue on
Mavericks](https://discussions.apple.com/thread/5475136) with a select
number of drives, including this one, apparently...

Common themes:

1. Started with OS X complaining at me because I supposedly ejected
the drive unsafely, even though the drive was still in the USB slot.
1. Drive isn't mounted automatically anymore. Console says filesystem
is busted: `17/12/13 20:18:35,000 kernel[0]: hfs: Runtime corruption
detected on My Passport, fsck will be forced on next mount.`
1. Drive can't be /unmounted/ using Disk Utility either both in
regular desktop mode and in recovery mode (booting with ⌘+R). This
means that even trying to just nuke the partition and starting over
doesn't work; it just sits there for a while and then eventually times
out saying it can't unmount the drive.

I did not, at any point, have WD's drive management software installed
(unless it managed to do it behind my back). First thing I did was put
a new GUID table on it with one partition.

Apparently the solution is to boot into Linux to fix the drive. I'll
let you know how that pans out.

UPDATE: Booted into Linux, was able to read everything. `fsck` does
report something wrong with the filesystem, but not bad enough that I
can't mount or umount it. Was able to salvage what looks to be all the
data I wanted; didn't bother to try and get e.g. Time Machine or
Spotlight index data off, which is where I suspect the issues to stem
from.
