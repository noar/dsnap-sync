<!-- dsnap-sync README.md -->
<!-- version: 0.5.5 -->

# dsnap-sync

## About

`dsnap-sync` is designed to backup btrfs formated filesystems.
It takes advantage of the specific snapshots functionality btrfs offers
and combines it with managemnet functionality of snapper.

`dsnap-sync` creates backups as btrfs-snapshots on a selectable target device.
Plug in and mount any btrfs-formatted device to your system. Supported devices
may be either local USB drives, but can be as well remote accessible RAID drives.
If possible the backup process will send incremental snapshots to the target drive.
If the snapshot will be stored on a remote host, it is secured with ssh.

The tool is implemented as a posix shell script, to keep the footprint small (dash).
`dsnap-sync` will support interactive and time scheduled backup runs.

## Backup process

For a backup run, and per default `dsnap-sync` will iterate through all defined snapper
configurations found on your source system. If you prefer to just run on a specific
configuration per call, you are free to select it using the 'config' option `-c`.

For each selected snapper configuration `dsnap-sync`

* will create an appropriate local snapshot and update the metadata
* will transfer the snapshot using btrfs-send to the target device
* will create and update the snapper configuration on the target
* will update the metadata on the target device

Usualy tools will document this proccess as a disk to disk (d2d) backup.
If possible `dsnap-sync` will levarage btrfs-send capabilities to only
send deltas. It will compare the snapshot data of the ongiong process with available
snapshot data on the target device.

### Interactive backups

An interactive run will request you to select a mounted btrfs device.
You can pre-select the target drive via [command line options](https://github.com/rzerres/dsnap-sync#options).
Either use a UUID, a SUBVOLID or a TARGET name (read 'mount point').

### Scheduled backups

A scheduled run will take all needed parameters from config options.
`dsnap-sync` does support systemd.timer units. Please refer to related paragraph [documenting systemd](https://github.com/rzerres/dsnap-sync#systemd).

## Requirements
beside the shell itself, `dsnap-sync`relies on external tools to achieve its goal.
At run-time their availability is checked. Following tools are are used:

- awk
- btrfs
- findmnt
- sed
- snapper
- tee
- wc

optionaly tools

- notify-send
- pv

## Installation

    # make install

If your system uses a non-default location for the snapper
configuration file, specify it on the command line with
`SNAPPER_CONFIG`. For example, for Arch Linux use:

    # make SNAPPER_CONFIG=/etc/conf.d/snapper install

The local snapper configuration will be extended to make use
of a new template 'dsnap-sync'.

The package is also available in the
<!-- [AUR](https://aur.archlinux.org/packages/dsnap-sync/). -->

## Options

    Usage: dsnap-sync [options]

    Options:
    -b, --backupdir <prefix>    backupdir is a relative path that will be appended to target backup-root
    -d, --description <desc>    Change the snapper description. Default: "latest incremental backup"
        --label-finished <desc> snapper description tagging successful jobs. Default: "dsnap-sync backup"
        --label-running <desc>  snapper description tagging active jobs. Default: "dsnap-sync in progress"
        --label-synced <desc>   snapper description tagging last synced jobs.
                                Default: "dsnap-sync last incremental"
        --color                 Enable colored output messages
    -c, --config <config>       Specify the snapper configuration to use. Otherwise will perform for each snapper
                                configuration. Can list multiple configurations within quotes, space-separated
                                (e.g. -c "root home").
        --config-postfix <name> Specify a postfix that will be appended to the destination snapper config name.
    -n, --noconfirm             Do not ask for confirmation for each configuration. Will still prompt for backup
        --batch                 directory name on first backup"
        --nonotify              Disable graphical notification (via dbus)
        --nopv                  Disable graphical progress output (disable pv)
    -r, --remote <address>      Send the snapshot backup to a remote machine. The snapshot will be sent via ssh.
                                You should specify the remote machine's hostname or ip address. The 'root' user
                                must be permitted to login on the remote machine.
    -p, --port <port>           The remote port.
    -s, --subvolid <subvlid>    Specify the subvolume id of the mounted BTRFS subvolume to back up to. Defaults to 5.
    -u, --uuid <UUID>           Specify the UUID of the mounted BTRFS subvolume to back up to. Otherwise will prompt."
                                If multiple mount points are found with the same UUID, will prompt user."
    -t, --target <target>       Specify the mountpoint of the BTRFS subvolume to back up to.
        --remote <address>      Send the snapshot backup to a remote machine. The snapshot will be sent via ssh. You
                                should specify the remote machine's hostname or ip address. The 'root' user must be
                                permitted to login on the remote machine.
        --dry-run               perform a trial run where no changes are made.
    -v, --verbose               Be more verbose on what's going on (use multiple times to be more verbose).
        --version		        show program version

## First run

If you have never synced to the paticular target device (first run), `dsnap-sync`
will take care to create the necessary target file-structure to store the snapshot.
As an option you can prepend a backup-path.

Before the sync job is started, source and target locations will be presented.
You have to confirm any further operation, or use defaults (option: noconfirm).

## Example command line usage

### dsnap-sync to local target

#### Default: no selections, run for all snapper configs

    # dsnap-sync

#### Default: Select two configs, the backupdir and verbose output

    # dsnap-sync --verbose --config root --config data2 --backupdir=toshiba_r700

#### Dry-run: Select config, select Target, as batchjob (--noconfirm)

    # dsnap-sync  -c root -s 265 --noconfirm --dry-run

### dsnap-sync to remote host

`dsnap-sync` will rely on ssh access to the target host. For batch usage make sure, that your
public key is accepted for remote login as user 'root'. You may have to adapt /root/.ssh/authorized_keys
on the target host.

On your target host, you should also verify the availability of a dsnap-sync config-template for snapper.
A template `dsnap-sync` is included in the package for your convenience.

#### Dryrun: Select remote host <ip/fqdn>, interactive, run for all configs

    dsnap-sync --dry-run --remote 172.16.0.3
	Selecting a mounted BTRFS device for backups on 172.16.0.3.
	  0) / (uuid=5af3413e-59ea-4862-8cff-304afe25420f,subvolid=257,subvol=/root)
	  1) /.snapshots (uuid=5af3413e-59ea-4862-8cff-304afe25420f,subvolid=258,subvol=/@snapshots-root)
	  2) /data2 (uuid=62a45211-9197-4a5f-aeaf-0ab803a42c32,subvolid=261,subvol=/data2)
	  3) /home (uuid=62a45211-9197-4a5f-aeaf-0ab803a42c32,subvolid=258,subvol=/home)
	  4) /data2/.snapshots (uuid=62a45211-9197-4a5f-aeaf-0ab803a42c32,subvolid=262,subvol=/@snapshots-data2)
	  5) /home/.snapshots (uuid=62a45211-9197-4a5f-aeaf-0ab803a42c32,subvolid=259,subvol=/@snapshots-home)
	  6) /var/lib/machines (uuid=2ba04452-74aa-44df-b1c7-74e0a70c6543,subvolid=260,subvol=/machines)
	  7) /var/lib/libvirt (uuid=2ba04452-74aa-44df-b1c7-74e0a70c6543,subvolid=261,subvol=/libvirt)
	  8) /data (uuid=2ba04452-74aa-44df-b1c7-74e0a70c6543,subvolid=257,subvol=/data)
	  9) /var/lib/machines/.snapshots (uuid=2ba04452-74aa-44df-b1c7-74e0a70c6543,subvolid=2121,subvol=/@snapshots-machines)
	 10) /data/.snapshots (uuid=2ba04452-74aa-44df-b1c7-74e0a70c6543,subvolid=258,subvol=/@snapshots-data)
	 11) /var/lib/dsnap-sync (uuid=753eba7a-41ce-49e0-b2e3-24ee07811efd,subvolid=420,subvol=/dsnap-sync)
	  x) Exit
    Enter a number: 11


### Dry-run with given Target for snapper config 'home', no confirmations

#### Sync: Select config 'data2', remote host <ip/fqdn>, target '/data', as batchjob (--noconfirm)

    # dsnap-sync --config data2 --remote 172.16.0.3 --target /data --noconfirm

## systemd

### service

    [Unit]
    Description=Run dsnap-sync backup

    [Install]
    WantedBy=multi-user.target

    [Service]
    Type=simple
    ExecStart=/usr/bin/dsnap-sync --UUID 7360922b-c916-4d9f-a670-67fe0b91143c --subvolid 5 --noconfirm

### timer

    [Unit]
    Description=Run dsnap-sync weekly

    [Timer]
    OnCalendar=weekly
    AccuracySec=12h
    Persistent=true

    [Install]
    WantedBy=timers.target

## snapper template

	###
	# template for dsnap-sync handling
	###

	# subvolume to snapshot
	SUBVOLUME="/var/lib/dsnap-sync"

	# filesystem type
	FSTYPE="btrfs"

	# users and groups allowed to work with config
	ALLOW_USERS=""
	ALLOW_GROUPS="adm"

	# sync users and groups from ALLOW_USERS and ALLOW_GROUPS to .snapshots
	# directory
	SYNC_ACL="yes"

	# start comparing pre- and post-snapshot in background after creating
	# post-snapshot
	BACKGROUND_COMPARISON="yes"

	# run daily number cleanup
	NUMBER_CLEANUP="no"

	# limit for number cleanup
	NUMBER_MIN_AGE="1800"
	NUMBER_LIMIT="10"
	NUMBER_LIMIT_IMPORTANT="2"

	# use systemd.timer for timeline
	TIMELINE_CREATE="no"

	# use systemd.timer for cleanup
	TIMELINE_CLEANUP="no"

    # dsnap-sync as timer unit
    SNAP_SYNC_EXCLUDE="yes"

## Contributing

Help is very welcome! Feel free to fork and issue a pull request to add features or
tackle open issues. If you are requesting new features, please have a look at the
TODO list. It might be already on the agenda.

## Related projects

I did fork from Wes Barnetts original work. I was aiming merged it back.
Beside the fact that this version doesn't use any bashisms, Wes did let me know,
that he doesn't have the time to review the changes appropriately to make it a merge.
Anyone willing to do so is invided.

Until that date, i will offer this fork for the public. To overcome any name clashes
i renamed it to dsnap-sync.


## License

<!-- License source -->
[Logo-CC_BY]: https://i.creativecommons.org/l/by/4.0/88x31.png "Creative Common Logo"
[License-CC_BY]: https://creativecommons.org/licenses/by/4.0/legalcode "Creative Common License"

This work is licensed under a [Creative Common License 4.0][License-CC_BY]

![Creative Common Logo][Logo-CC_BY]

© 2016, 2017  James W. Barnett;
© 2017 - 2018 Ralf Zerres
