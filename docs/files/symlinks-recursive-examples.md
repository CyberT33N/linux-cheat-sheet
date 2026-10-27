# Recursive Symlink Batch Examples

Companion to [Symlinks](./symlinks.md). Three worked shell examples that create symlinks for all folders except a specific excluded folder, and copy the excluded folder so it is used without a symlink.

## Create symlincy recursive for all folders expect of specific folder

```shell
# example 1
MIGRATIONS_PATH="/home/dennis/Projects/migrations"

CUSTOM_PROJECT="test-10400"
CUSTOM_PROJECT_PATH="data/$CUSTOM_PROJECT"

INITIAL_PROJECT="initial-project"
INITIAL_PROJECT_PATH="data/$INITIAL_PROJECT"

EXLUDED_FOLDER="Apple"

create_symlinks() {
    for folder in ../$INITIAL_PROJECT/*; do
       if [[ ! "$folder" =~ "$EXLUDED_FOLDER" ]]; then
           ENTITY_NAME="$(basename -- "$folder")"
           echo "ENTITY_NAME: $ENTITY_NAME"
   
           SOURCE="../$INITIAL_PROJECT/$ENTITY_NAME"
           echo "SOURCE: $SOURCE"
     
           DESTINATION="./$ENTITY_NAME"
           echo "DESTINATION: $DESTINATION"  
       
           ln -s "$SOURCE" "$DESTINATION"
       fi
    done
}

# Navigate to the data directory
cd $MIGRATIONS_PATH

# Remove existing directory
rm -rf "$CUSTOM_PROJECT_PATH"

# Create the new client folder
mkdir $CUSTOM_PROJECT_PATH

# Create symlinks
(cd $CUSTOM_PROJECT_PATH; create_symlinks)

# Copy the excluded folder to use it without a symlink
cp -r "$INITIAL_PROJECT_PATH/$EXLUDED_FOLDER" "$CUSTOM_PROJECT_PATH/$EXLUDED_FOLDER"

# Verify that everything worked
ls -l "$CUSTOM_PROJECT_PATH/SchemaEntities"

# example 2
mkdir -p ~/Dropbox
cd ~
for file in *; do if [[ ! "$file" =~ Video|Dropbox ]]; then ln -s "$HOME/$file" "Dropbox/$file"; fi; done

# example 3
cd ~/lib/migrations/data

# create new client folder
mkdir "test-10400"

# cd into this folder because we will iterate of every file
cd "initial-project"

for file in *; do if [[ ! "$file" =~ cs.Apple ]]; then ln -s "$file" "../test-10400/$file"; fi; done

# Copy the excluded folder - So we use it without symlink
cp -r "cs.Apple" "../test-10400/cs.Apple"
```