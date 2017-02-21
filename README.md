# systemd-common

Common dropins for systemd units.

## Installation

All you need to do is clone this repo somewhere onto your disk. Typically, I clone this into my `/srv` directory.

```
git clone https://github.com/flungo/systemd-common.git /srv/common
```

## Dropins

Dropins are provided in the `dropins/` directory. To use the dropins, create the dropins folder in your `/etc/systemd/system` directory and create a symbolic link. For example, if you had the service `foo.service` and wanted to add the dropin `bar.conf` with systemd-common installed to `/srv/common` these are the steps you would take.

```
cd /etc/systemd/system
mkdir -p foo.service.d
ln -s /srv/common/dropins/bar.conf foo.service.d/bar.conf
```

Documented in this README, are the types and variants for dropins. Within each group, its recomended that they are linked with the same name, allowing them to be overriden by instances (i.e. the name that they are linked as should be descriptive of what they configure but not how its configured). The title of each section denotes the suggested name although you may wish to prepend numbers so that you can order the dropins within the service. Reffer to the [systemd.unit documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) for more information on dropins and how they work.

### restart-policy

The `restart-policy` dropins that are provided are:

- `restart-no` never restart the service when it stops.
- `restart-always` always restart the service when it stops.
- `restart-on-success` restart the service when it terminates successfully.
- `restart-on-failure` restart the service when it terminates with any failure condition.
- `restart-on-abnormal` restart the service when it terminates abnormally (unclean signal, timeout or watchdog but not an unclean exit code).
- `restart-on-abort` restart the service when its aborted by a signal.
- `restart-on-watchdog` restart the service when its terminated by the watchdog timer.

For more details on the possible restart policies, reffer to the [systemd.service Restart documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html#Restart=).
