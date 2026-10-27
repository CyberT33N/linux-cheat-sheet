# IP & Port Inspection

## Check what processes uses the specific port

```bash
lsof -i :30500
```

## Get adapter ip adresses

```bash
ifconfig
```

> Note: the source section contained the stray line `dejs/` between the port-inspection and IP recipes; it is preserved here as an explicit disclosure instead of being silently dropped (no-breaking policy). It carries no operational content and may be removed on request.