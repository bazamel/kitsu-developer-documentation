# Log rotation

Logs produced by the API grow quickly and lead to very large files. Configure
`logrotate` so a fresh log file is created every day and old ones are pruned
automatically.

::: info Docker users
If you run Kitsu through the official Docker image, logs go to the container's
stdout and are handled by your Docker logging driver — you don't need any of
the steps below. This guide is for bare-metal / systemd installs.
:::

## Store the PID of the zou processes

`logrotate` needs each Gunicorn process' PID so it can signal it to reopen its
log files after a rotation. Have systemd create a runtime directory on boot and
write the PIDs there.

Edit the zou unit file (`/etc/systemd/system/zou.service`) and add a
`RuntimeDirectory` line before `ExecStart`:

```ini
RuntimeDirectory=zou
```

Then append `-p /run/zou/zou.pid` to the `ExecStart` line, for example:

```ini
ExecStart=/opt/zou/zouenv/bin/gunicorn -p /run/zou/zou.pid -c /etc/zou/gunicorn.py -b 127.0.0.1:5000 zou.app:app
```

Do the same in the zou-events unit file
(`/etc/systemd/system/zou-events.service`), using a distinct PID file:

```ini
-p /run/zou/zou-events.pid
```

Reload systemd and restart the services so the changes take effect:

```bash
sudo systemctl daemon-reload
sudo systemctl restart zou zou-events
```

The PIDs are now written to `/run/zou/zou.pid` and `/run/zou/zou-events.pid`.

## Configure logrotate

`logrotate` is a standard Unix tool that rotates logs for you based on a
configuration file. Create `/etc/logrotate.d/zou`:

```
/opt/zou/logs/gunicorn_access.log /opt/zou/logs/gunicorn_error.log {
    daily
    missingok
    rotate 14
    notifempty
    nocompress
    size 100M
    create 644 zou zou
    sharedscripts
    postrotate
        kill -USR1 `cat /run/zou/zou.pid`
    endscript
}

/opt/zou/logs/gunicorn_events_access.log /opt/zou/logs/gunicorn_events_error.log {
    daily
    missingok
    rotate 14
    notifempty
    nocompress
    size 100M
    create 644 zou zou
    sharedscripts
    postrotate
        kill -USR1 `cat /run/zou/zou-events.pid`
    endscript
}
```

What the directives do:

| Directive | Effect |
|-----------|--------|
| `daily` | Rotate once a day |
| `rotate 14` | Keep the last 14 rotated files |
| `size 100M` | Also rotate as soon as a file exceeds 100 MB |
| `missingok` | Don't error if a log file is absent |
| `notifempty` | Skip rotation when the file is empty |
| `nocompress` | Keep rotated files uncompressed (set `compress` to gzip them) |
| `create 644 zou zou` | Recreate the log with these permissions and owner |
| `sharedscripts` | Run `postrotate` once for the whole block, not per file |
| `postrotate … kill -USR1` | Tell Gunicorn to reopen its log files |

::: tip Save disk space
Replace `nocompress` with `compress` to gzip rotated logs. This trades a little
CPU at rotation time for much smaller archives.
:::

## Verify

Dry-run the configuration to confirm it is picked up correctly:

```bash
logrotate /etc/logrotate.d/zou --debug
```

You're done with log rotation!
