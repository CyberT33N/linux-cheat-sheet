# Navigation & Redirection

## cd

### Go to parrent directory

```bash
cd ..
```

### Go to last directory

```bash
cd -
```

### Go to home

```bash
cd
```

### Change to current directory of folder where the script file located

```bash
cd "$(dirname "$0")"
pwd
printf "\nWe will display now the current directory used:" && pwd
```

## cat

### Combine content of 2 files to new file

```bash
cat test1.txt test2.txt > test3.txt
```

## Pipe

- Pipe the results of any command to a new file

```bash
ls > result.txt
```

### Add new text to file

```bash
echo hi >> result.txt
```

### Pipe logs into file that you do not see stdout

```bash
port-forward.sh > /dev/null & echo 'Do other stuff where you do not want to see stdout of port-forward.sh'
```