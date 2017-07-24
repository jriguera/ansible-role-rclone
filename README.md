# ansible-role-rclone

A versatile role to setup rclone, created using this philosophy: 
https://github.com/jriguera/ansible-role-pattern/blob/master/README.md

Ansible 2.3, works with Ubuntu Xenial and Centos 7 (in general with any systemd distribution)

This role supports the definition of different backups folders and parameters
defined as systemd services and triggered by systemd timers.

The idea as create a sort of rclone backup agent completely managed by systemd
timers (instead of cron jobs). It is also capable of setting Xwindow notifications
using `notify-send` utility.

## Configuration

Have a look at the default configuration parameters defined in `default/main.yml`

This is an example for a backup configuration:

```
rclone_version: "1.36"
rclone_config_file: "/etc/rclone.conf"

# Configuration settings definition dictionary. You can copy it directly from
# the rclone config file (I know, it requires you to setup it manually first
# in order to get these parameters).
rclone_config:
  AmazonCloud:
   type: "amazon cloud drive"
   client_id: null
   client_secret: null
   token: '{"access_token":"balblablaa","expiry":"2017-05-13T17:19:52.022195275+02:00"}'

# Backup definition settings
rclone_backup_timer_enable: true
rclone_backup_timer_state: started

# run rclone every hour
rclone_backup_timer_cron: "0/1"
rclone_backup_boot_delay: "15min"

# Folder backup definition list. Each item has
#
# enable: Enable systemd service, defaults to true
# delete: Delete all files created, defaults to false
# destination: Rclone configuration destination name
# operation: Rclone command, defaults to sync
# folder: Directory where to perform backups from
# args: Additional arguments for the command, defaults to --copy-links
# osd_notify_user: enabled osd notifications to the specified user
# (you need X system running!)
#
rclone_backups:
- destination: "AmazonCloud"
  folder: "/home/jriguera"
  args: "-v --copy-links --one-file-system"
  osd_notify_user: jriguera
  filters:
  - "- .Private/"
  - "- .atom/"
  - "- .adobe/"
  - "- .mozilla/"
  - "- .macromedia/"
  - "- .gvfs/"
  - "- .gnome2_private/"
  - "- .cache/"
  - "- .dotfiles/"
  - "- .ecryptfs/"
  - "- .bundle/"
  - "- .bosh/"
  - "- .bosh_init/"
  - "- .dropbox*/"
  - "- .glide/"
  - "- .thumbnails/"
  - "- .vagrant.d/"
  - "- .gem/"
  - "- .ansible/"
  - "- .bundle/"

```

With this configuration, it will create a `.backup.conf` file in `/home/jriguera`
with all those settings. It will also create a `.backup.filters` in the same folder.
Please be aware of the filter definition uses the format defined in
https://rclone.org/filtering/#filter-from-read-filtering-patterns-from-a-file

This Ansible role uses systemd timers (not cron), so:

* List the systemd timers, type: `systemctl list-timers --all`
* To stop rclone timer: `systemctl stop rclone.timer`
* To disable rclone timer: `systemctl disable rclone.timer`
* To run manually the backups (with the timer stopped/disabled): `systemctl start rclone.target`
* To list the systemd rclone units: `systemctl list-unit-files rclone*`

For each backup, it will create a service instance (with `@`) which can be
independently enabled or disabled, inheriting from the rclone target definition.


## Author

José Riguera López <jriguera@gmail.com>  2017

