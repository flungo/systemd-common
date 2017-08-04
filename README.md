# systemd-common

Common components for use with systemd.

## Installation

Installation of these common files requires that the repo is copied to the disk. To ensure that all units work as expected it is recommended that the repository is cloned into `/srv/common`.

```
git clone https://github.com/flungo/systemd-common.git /srv/common
```

Individual files can then be installed using symbolic links as described in the relevant sections below.

## Dropins

Dropins are provided in the `dropins/` directory. To use the dropins, create the dropins folder in your `/etc/systemd/system` directory and create a symbolic link. For example, if you had the service `foo.service` and wanted to add the dropin `bar.conf` with systemd-common installed to `/srv/common` these are the steps you would take.

```
cd /etc/systemd/system
mkdir -p foo.service.d
ln -s /srv/common/dropins/bar.conf foo.service.d/bar.conf
```

Documented in this README, are the types and variants for dropins. Within each group, its recomended that they are linked with the same name, allowing them to be overriden by instances (i.e. the name that they are linked as should be descriptive of what they configure but not how its configured). The title of each section denotes the suggested name although you may wish to prepend numbers so that you can order the dropins within the service. Reffer to the [systemd.unit documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) for more information on dropins and how they work.

Where multiple dropins are contained within a directory, these dropins should be used mutually exclusive of each-other and the directory name denotes the name with which the dropin should be linked.

For other dropins, a template is provided. These templates end with `.j2` and are written using the [Jinja2 templating language](http://jinja.pocoo.org/). These should be used with Jinja to produce the relevant dropin.

### restart-policy

The `restart-policy` dropins that are provided are:

- `restart-no` never restart the service when it stops.
- `restart-always` always restart the service when it stops.
- `restart-on-success` restart the service when it terminates successfully.
- `restart-on-failure` restart the service when it terminates with any failure condition.
- `restart-on-abnormal` restart the service when it terminates abnormally (unclean signal, timeout or watchdog but not an unclean exit code).
- `restart-on-abort` restart the service when its aborted by a signal.
- `restart-on-watchdog` restart the service when its terminated by the watchdog timer.

All of the provided policies set `RestartSec=5` so that the service will sleep for 5 seconds between restarts.

For more details on the possible restart policies, refer to the [systemd.service Restart documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html#Restart=).

### onfailure-email-status

Using this dropin will result in an email being sent using the `email-status@.service` unit when the unit which this dropin is used with fails. See the unit details to be able to configure the email address which the notification is sent to.

## Units

Units provide services which can be used with timers, hooks or independently. To install the units they should be symbolically linked to from the `/etc/systemd/system` directory.

For some units, dropins are provided in a directory using the same name as the unit they are provided for suffixed with `.d`. For example, dropins for the `email-status@.service` are provided in the `email-status@.service.d` directory. These dropins folders follow the same format as those found in the global [dropins](#dropins) directory.

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

### email-status

A unit which emails the status of another services using the email-status script is included. When run with a service pattern escaped with `systemd-escape` as the instance name, the status of matching services are sent in an email to the email address specified by the `RECIPIENT_EMAIL` environment variable. By default, the status is sent to `root`. In order to have the message delivered to another user or an email address (providing the MTA on the host machine is configured to deliver to that user and/or domain), a dropin which sets the `RECIPIENT_EMAIL` environment variable should be added. A template dropin for this has been provided as `units/email-status@.service.d/templates/recipient-email.conf`.

By using instance dropins, a different email can be used for specific services. For example, to mail the `webmaster` user when `nginx.service` fails, then a dropin setting the `RECIPIENT_EMAIL` environment variable can be placed in `/etc/system/systemd/email-status@nginx.service.d/`.

## scripts

In order to assist with some units, scripts have been provided. These are expected to exist within `/srv/common/scripts` and so if this repo has been installed elsewhere, it will be required to make modified copies of the units provided.

### email-status

The `email-status` script sends an email to the provided address containing the status of the services which match the pattern provided to the email address provided. It uses the `sendmail` application to send the email and so requires that this is installed and configured correctly. This is designed to be used with [`OnFailure=`](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#OnFailure=) (and [a dropin](#onfailure-email-status) is provided to assist with this) but can also be setup with a timer to regularly send status updates.

```
email-status RECIPIENT PATTERN
```

- **RECIPIENT** The email address of the recipient.
- **PATTERN** The pattern for units to report the status on.
