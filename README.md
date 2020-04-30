# smartarray-scripts
SmartArray RAID controller scripts for monitoring and diagnostics.

The cciss* family of scripts should work on any controller driven by
the cciss or hpsa driver as long as the appropriate utility is present.

The hpsa* family of scripts work only on controllers driven by the
hpsa driver.

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

## cciss-set-surfacescan

Script to easily change surfacescanmode on all HP/HPE SmartArray controllers
in a machine.  The surface scan process periodically verifies that all blocks
on all drives are readable.

You want to fiddle with this when:

* You have a workload that always hammers the drives preventing surface scan to run, and you want to automate setting it to high during low-load times (weekends for example).
* You are running SMART long selftests and wondering why they never finish.

### Usage examples

This is a cut&paste of the ACC `/etc/cron.d/hp-smartarray-surfacescan`
cron job run on machines which have a small redundant LD for the OS
and a large non-redundant LD for caching data:

```
# HP SmartArray controllers has a surface scan feature, BUT it only seems
# to scan redundant logical drives. To work around this we kick regular
# SMART selftests using smartmontools (via a helper script).

# Disable HP SmartArray surface scan and trigger SMART selftest.

# Do short selftest during weekdays
10 4 * * 1-5 root cciss-set-surfacescan disable > /dev/null ; hpsa-smartctl -t short -q silent
# Set surface scan mode to our default to scan only when idle during weekdays
10 5 * * 1-5 root cciss-set-surfacescan idle > /dev/null

# NOTE: Running a long self test (ie. surface scan) frequently is a BAD IDEA on
# drives with a low workload rating (ie Midline/7kRPM drives).  We assume that
# the long selftest will run/complete over the weekend.
10 1 * * 6 root cciss-set-surfacescan disable > /dev/null ; hpsa-smartctl -t long -q silent

# Check all physical drives for SMART failures, the Smart Array controller
# should report these as predictive failures but better safe...
12 5 * * * root hpsa-smartctl -H -l selftest -q errorsonly || echo "SmartArray drive with SMART errors detected."
```
