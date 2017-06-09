# systemd-common

Common components for use with systemd.

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

## Units

Units provide services which can be used with timers, hooks or independently. To install the units they should be symbolically linked to from the `/etc/systemd/system` directory.

### systemctl

Units which can be used to control services by stopping, starting and restarting them have been provided. These are best used with timers so that these tasks can be performed on a schedule. The units provided are:

- `systemctl-restart@.service`
- `systemctl-start@.service`
- `systemctl-stop@.service`

For each unit, the instance name should be the escaped pattern of the service for the action to be performed on, as escaped by `systemd-escape`. For example to restart the `nginx@example.com` service daily, the following timer unit can be created as `systemctl-restart@nginx\x40example.com.timer`:

```
[Unit]
Description=Restart nginx@example.com daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

**Note:** Make sure that when creating files and running commands that the `\` character is not treated as a shell escape character by including it in single quoted strings or by escaping the backslash.
