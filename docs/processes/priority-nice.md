# Process Priority (nice)

## Set priority to process

```shell
# nice -%LEVEL% your-app
# Where %LEVEL% is a value of “niceness” from -19 (maximal priority, minimal niceness) to 20 (minimal priority). For “niceness” less than zero you have to use root rights. For -19 niceness use:

sudo nice --19 your-app
```