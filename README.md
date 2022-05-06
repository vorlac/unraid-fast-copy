# unraid-fast-copy

An optimized parallel copy utility for copying data from an unraid share to a mounted nfs/cifs share


## Overview

This script is stil a work in progress, so at the moment all parameters must be manually defined at the top of the script.

Each parameter has a descriptive comment above it that outlines how the primary values should be set based on:
1. The unraid share you want to copy, optionally including a subdirectory path relative to the root of the share if you want to scope the copy to just a portion of the share.
2. The target destination of the copy, which should be a locally mounted smb or nfs share using the current implementation of this script.

The scipt will first determine which disk shares contain data from the share (including subdirectory if provided). Then for each disk that contains share data, one rsync process will be spawned to recursively copy the share data direclty from the disk mount. This allows each individual process to read data from each disk as fast as possible while not contending for resources from the same phsyical disk.


## Dependencies

Any unraid app/plugin that enables the rsync command within the unraid shell is the only thing required to get this script running.

Once that's installed, just fill in the script parameters, then add execute permissions to the script:
```bash
chmod +x ./unraid-fast-copy.sh
```
then just simply call the scipt to execute the copy:
```bash
./unraid-fast-copy.sh
```

## Results

With an array of 12x spindle disks (2 parity disks + 10 data disks) and no ssd/ram cache I was able to get copy speeds slightly over 1 GB/s peak, but avereaged out to a pretty consistent ~800MB/s when copying a ~75TB share that spanned all 10 data disks on an ancient Dell R510 server. With the latest script changes I started getting bottlenecked by the 10G network link and/or remote share server's write speeds.

IO wait times reported by netdata are now down to just ~10-15% compared to the 50-75%+ iowait I saw when copying directly from the user share mount with a few different approaches.



# TODO:

- Add command line arg parser
- Validate and sanitize input paths for the source and target directories to avoid issues by including/excluding preceeding or trailing slashes (which is something to keep an eye out for now, read the comments in the script for more info).
- Imrpove the `print_progress` function to eventually provide the status of each child rsync process while the script is running (a prototype of this function exists, but is buggy and currently commented out)
   - Get it to properly handle terminal resizing
   - Monitor active processes by polling for their activity to further improve what type of progress can be printed
   - Update the main loop so it exits when all child processes are finished copying the data for each disk that contains share data.
 - Add dry-run mode that just generates a report of what the script plans to do without copying/modifying anything.