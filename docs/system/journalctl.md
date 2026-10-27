# Journalctl — System Logs

- If you want to find logs after you system crashed try to search for `-- Boot`

## Check logs realtime

```bash
sudo journalctl -f
```

## Check logs specific time

```bash
journalctl --since "2021-11-30 15:00" --until "2021-11-30 17:00" > ~/log.txt
```

## Check logs 10 Minutes ago

```bash
journalctl --since "10min ago" > ~/log.txt
```

## Check logs specific time from NetworkManager service

```bash
journalctl -u NetworkManager.service --since "2021-11-30 15:00" --until "2021-11-30 17:00" > ~/log.txt
```

## Get system logs to find problems as example when system crash

```bash
# -1 last restart | -2 last restart before 2 days
journalctl -o short-precise -k -b -2

# Search logs that only contains "error"
journalctl -o short-precise -k -b -2 | grep -i "error"
```