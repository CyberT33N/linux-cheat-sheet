# Terminal Configuration

## Buffer size

### Konsole

- If you have started a shell session in Konsole, do the following:

Konsole > Settings > Edit current profile > Scrolling > Scrollback

## Open script in new terminal window and do not close it when its done

```bash
xterm -geometry 93x31+100+350 -xrm 'XTerm.vt100.allowTitleOps: false' -T Minikube -hold -e $SHELL -c "bash ./setup-port-foward.sh" &

xterm -geometry 93x31+100+750 -xrm 'XTerm.vt100.allowTitleOps: false' -T Minikube -hold -e $SHELL -c "bash ./setup-port-foward.sh" &
```