# Symlinks

- You can create symlinks for files and folders
  - If you decide to symlink a folder then the folder for your destination must not exists because it will be automatically created

- You can use absolute or relative paths. However for git projects you should always use relative paths that other developers can cooperate with your symlinks inside the project
  - It is very important that you cd inside the the destination directory where you want to create the symlink for your folder / file

## Create symlink to another file to use it.

```bash
ln -s ../project/.eslintrc.json ./.eslintrc.json
```

### Example #2

```bash
repoRoot="$HOME/Projects/cheat-sheets/ai/prompts-cheat-sheet"

# Source (the file you actually maintain)
sourcePath="$repoRoot/IDE/prompts/creation-prompts/master-prompts.md"

# Link (the "Architecture" file inside .cursor/commands)
linkDir="$HOME/.cursor/commands"
linkName='[PROMPT] - [PROMPTING] - MCP.md'
linkPath="$linkDir/$linkName"

# If the link path exists (including broken symlinks), remove it
if [[ -e "$linkPath" || -L "$linkPath" ]]; then
  rm -f -- "$linkPath"
fi

# Create the symlink
ln -s -- "$sourcePath" "$linkPath"
```

## Create a link to another folder to use it.

```bash
# In this case
# ln -s FolderWhereYouCentralFiles FolderWhereYouCreateSymlink
ln -s ~/Projects/ai/checkpoints/Stable-diffusion ~/Projects/stable-diffusion-webui/models
ls -l ~/Projects/stable-diffusion-webui/models/Stable-diffusion
```

## Show existing symlinks

```bash
ls -la /path/here
```

## Remove symlink

```shell
rm -i /pfad/zum/zielordner
```