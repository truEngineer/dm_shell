# Data Mining Shell

## Remove duplicate files
zsh:
```bash
md5 -r  * | sort | awk 'BEGIN{lasthash = ""} $1 == lasthash {print $2} {lasthash = $1}'
```
