#btrfs-balance-slowly.python

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

This script re-runs the balance operation until it finds no further work to do.
In between each run (called a "portion"), the script sleeps for a while, to allow the
system to perform other work.
Also, commands can be executed before and after each portion run to allow time-sensitive services
such as mail to be stopped during the balance.

## Usage

```
btrfs-balance-slowly.python  [options] <disk>
        <disk>          disk to balance, for example: /mnt/backup2
Options:
	-u | --usage		<percentage> - only consider block groups with less than this percentage used
        -l | --limit   		<count> - maximum number of portions, default 0 (no limit)
        -t | --time     	<seconds> - maximum duration of the whole operation (in seconds), default 3600
	-L | --portion-limit	<count> - maximum number of block groups in a portion, default 0 (no limit)
        -T | --portion-time    	<seconds> - maximum duration for one portion (in seconds), default 3600
        -i | --interval		<seconds> - delay between portions (in seconds), default 600
        -H | --hook     	<command-or-function> - python function to execute before/after portion of balance, default``hook_null``
	-h | --help		display this help
```

## Hooks
The python function specified with --hook is called before and after each portion with 2 arguments:
```
hook (PRE, <disk>)
hook (POST, <disk>)
```
``PRE`` and ``POST`` are constants to indicate which context the hook is being called for.
``<disk>`` is a string containing the disk specified on the command line.

The function must raise an exception if processing is not to continue.

There are two pre-defined hook functions: ``hook_null`` and ``hook_nomail``.
``hook_null`` does nothing, and is the default.

``hook_nomail`` is specific to my particular use: it stops fetchmail, postfix and dovecot during the run
and restarts them afterwards. It may provide a useful example for creating other hook functions.

## Example
Here is a real example taken from my cron jobs (but with line breaks added for readability)
```
4 5 * * thu       root    /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -musage=0,limit=10 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -dusage=0,limit=5 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --time 600 --interval 300 -musage=20,limit=10 /mnt/data ;
 /usr/local/sbin/btrfs-balance-slowly --limit 20 --time 3600 --hook hook-nomail -dusage=20,limit=5 /mnt/data
```

## Notices
Copyright (C) 2019 Graham R. Cobb
Copyright (C) 2017 Hans van Kranenburg <hans@knorrie.org>

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Note: this software is based on btrfs-balance-least-used, by
Hans van Kranenburg <hans@knorrie.org>. Used with permission.

#btrfs-scrub-slowly

Similar script for btrfs scrub commands.

Although not as disruptive as balance, scrubs can cause big slowdowns.

I normally use this with long time limits set but meaning that if it gets too slow I can
manually cancel the scrub knowing that the script will resume it in a while.

Usage and hooks work the same as btrfs-balance-slowly except that no <filter> is used.

## Example
Real example from my cron jobs:
```
40 2 1 * *    root    /usr/local/sbin/btrfs-balance-slowly --portion-time $((6*60*60)) --interval $((1*60*60)) /mnt/data/
```

## Notices
Copyright (c) 2016-2018 Graham R. Cobb.
This software is distributed under the GPL (see the copyright notices and the LICENSE.gpl2 file).

`btrfs-scrub-slowly` is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

`btrfs-scrub-slowly` is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

#btrfs-balance-slowly

This is the original bash version of the script, no longer maintained. The python version (above) is now the preferred version.

The information below is provided for reference.

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
Copyright (c) 2016-2018 Graham R. Cobb.
This software is distributed under the GPL (see the copyright notices and the LICENSE.gpl2 file).

`btrfs-balance-slowly` is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

`btrfs-balance-slowly` is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
