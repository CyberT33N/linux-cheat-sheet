# I/O Limit For An Application Via Systemd User Service — Activation, Logs And Operations

Part of the systemd I/O-limiting guide; see also [physical device detection](./physical-device-detection.md) and [service configuration](./systemd-service-configuration.md).

## 4. Activate and start the service

```bash
systemctl --user daemon-reload
systemctl --user start cursor-throttled.service
```

Optional autostart:

```bash
systemctl --user enable cursor-throttled.service
```

## 5. Check logs

```bash
journalctl --user -u cursor-throttled.service -f
```

## ⚠️ Important

* **I/O limiting only works with the real physical device (Major\:Minor)!**
* With encrypted partitions you have to find the real device, not the mapper.
* The value for `IOReadBandwidthMax` and `IOWriteBandwidthMax` is a combination of Major\:Minor and bandwidth (e.g. `249:3 1M` for 1 megabyte per second).
* The system must use `cgroups v2` (Ubuntu 20.04+ with newer kernels).

## Change values and restart with:

```shell
systemctl --user daemon-reload
systemctl --user restart cursor-throttled.service
```

## Disable the autostart completely:

```bash
systemctl --user disable cursor-throttled.service
```

## Stop the currently running service:

```bash
systemctl --user stop cursor-throttled.service
```