# Process Management

## Kill process

```bash
# To kill process in Linux with PID:
kill pid

# To kill process in Linux with application name:
killall appname
```

## Force kill process

```bash
# To kill process in Linux with PID:
kill -9 pid

# To kill process in Linux with application name:
killall -9 appname
```

## Kill all processes on specific port

```bash
sudo kill -9 $(sudo lsof -t -i:30500)
```

## Get details about user processes

```shell
ps -u userHere -f
```

## Get details about process tree

```shell
pstree -p | grep mongosh
```