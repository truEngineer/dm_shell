# Data Mining Shell

## Remove duplicate files

The `awk` part initializes `lasthash` with the empty string, which will not match any hash, and then checks for each line if the hash in `lasthash` is the same as the hash (first column) of the current file (second column). 
If it is, it prints it out. At the end of every step it will set `lasthash` to the hash of the current file.

```bash
$ ls
train_1.jpeg	train_3.jpeg	train_4.jpeg	train_5c.jpeg
train_2.jpeg	train_3c.jpeg	train_5.jpeg	train_5cc.jpeg
$ md5sum * | sort | awk 'BEGIN{lasthash = ""} $1 == lasthash {print $2} {lasthash = $1}'
train_3c.jpeg
train_5c.jpeg
train_5cc.jpeg
```

The filenames `awk` spits out are fed to `rm` with `xargs`, which basically calls `rm` with what the `awk` part gives us.

🐧:
```bash
md5sum * | sort | awk 'BEGIN{lasthash = ""} $1 == lasthash {print $2} {lasthash = $1}' | xargs rm
```
🍏:
```bash
md5 -r * | sort | awk 'BEGIN{lasthash = ""} $1 == lasthash {print $2} {lasthash = $1}' | xargs rm
```

## Renaming multiple files

Renaming files in a folder to sequential zero padded numbers

🐧|🍏:
```bash
ls *.jpeg | cat -n | while read n f; do mv "$f" `printf "%03d.jpg" $n`; done
```
