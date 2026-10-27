# Cronjob

## List all cronjobs

```bash
crontab -l
```

## Create new cronjob

```bash
crontab -e
1 * * * * /home/usernamehere/Projects/start.sh
```

## Remove cronjob

```bash
// Text editor inside of terminal will open. Here you can remove your cronjobs
crontab -e
```