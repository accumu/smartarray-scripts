# smartarray-scripts
SmartArray RAID controller scripts for monitoring and diagnostics

## hpsa-smartctl

Helper-script to issue smartctl commands to all disks connected
to HP/HPE Smart Array controllers using the hpsa driver (ie. Px1x and newer).

This script is most useful for setups that use RAID 0 (ie. non-redundant)
logical drives, as the SmartArray controllers never will flag those drives
as bad but instead happily passes through all errors to the OS.

Requires `smartctl` and `lsscsi` in path.

### Usage examples

`hpsa-smartctl -a` - Check all status of all drives.

`hpsa-smartctl -H -l selftest -q errorsonly || echo "SmartArray drive with SMART errors detected."` - Emit a short error message in a cron job if a faulty drive is found.

`hpsa-smartctl -t short` - Start a short selftest (see note below on surface scans).

### Interaction between SMART selftests and SmartArray surface scans

Experience shows that SmartArray surface scans has higher priority than the
SMART selftests, effectively blocking SMART selftests from finishing.

See the `cciss-set-surfacescan` script on how to automate disabling/enabling
surface scans in order to work around this.
