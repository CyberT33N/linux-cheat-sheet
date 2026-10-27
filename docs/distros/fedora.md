# Fedora

## Show if package with specific name exist

```bash
sudo dnf list | grep "dragondisk"
```

## ~/bin

- In this folder everything is global. This means you can use everything global in your terminal

- If your content of the bin folder is not available in your terminal add to your .bashrc or .zshrc

```shell
export PATH="$HOME/bin:$PATH"
```