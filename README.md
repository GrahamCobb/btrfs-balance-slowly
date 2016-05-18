#btrfs-balance-slowly

Run btrfs balance with frequent breaks.

btrfs balance on my main data disk completely disrupts the system.
It runs for several hours and most access to the disk just hangs.
Even creating a new file using "touch" can hang for over an hour while a balance
is in progress.
Many services report timeouts and errors.
In particular, this completely destroys mail processing -- mail handling times out,
which is often treated as a permanent error and mails are lost.

The balance "limit" filter is useful in reducing the problem
(although a time-based filter instead of a count-based filter would be much more useful).
However, on a heavily used disk, the filtered balance has to be re-run many times.

This script re-runs the balance command until it finds no further work to do.
In between each run (called a "portion"), the script sleeps for a while, to allow the
system to perform other work.
Also, commands can be executed before and after each run to allow time-sensitive services
such as mail to be stopped during the balance.

## Usage

```
btrfs-balance-slowly  [options] <filter> <disk>
        <filter>        balance filter, for example: -dusage=20,limit=10
        <disk>          disk to balance, for example: /mnt/backup2
Options:
        -l | --limit    <count> - maximum number of portions, default 0
        -t | --time     <seconds> - maximum time (in seconds), default 3600
        -i | --interval <seconds> - delay between portions, default 600
        -h | --hook     <command-or-function> - command to execute before/after portion of balance, default "true"
```

## Hooks
The command specified with --hook is called before and after each portion with 3 arguments:
```
hook "pre" <filter> <disk>
hook "post" <filter> <disk>
```

The command must exit with success (0) if processing is to continue.
An error return will cause processing to terminate.

There is one pre-defined hook function: ``hook-nomail``.
This is specific to my particular use: it stops fetchmail, postfix and dovecot during the run
and restarts them afterwards.

## Example
Here is a real example taken from my cron jobs (but with line breaks added for readability)
```
4 5 * * thu       root    /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -musage=0,limit=10 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -dusage=0,limit=5 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -musage=20,limit=10 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --limit 20 --time 3600 --hook hook-nomail -dusage=20,limit=5 /mnt/data
```

## Notices
Copyright (c) 2016 Graham R. Cobb.
This software is distributed under the GPL (see the copyright notices and the LICENSE file).

`btrfs-balance-slowly` is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

`btrfs-balance-slowly` is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
