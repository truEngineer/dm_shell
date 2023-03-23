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

Renaming files in a folder to sequential zero padded numbers (`printf` is used for padding).

```bash
$ ls *.jpeg | cat -n
     1	train_a.jpeg
     2	train_b.jpeg
     3	train_c.jpeg
     4	train_d.jpeg
     5	train_e.jpeg
     6	train_f.jpeg
     7	train_g.jpeg
     8	train_h.jpeg
$ ls *.jpeg | cat -n | while read n f; do mv "$f" `printf "%03d.jpg" $n`; done
$ ls *.jpg
001.jpg	002.jpg	003.jpg	004.jpg	005.jpg	006.jpg	007.jpg	008.jpg
```

A **backtick \`** is not a quotation sign! Everything you type between backticks is evaluated (executed) by the shell before the main command (like `mv`), and the output of that execution is used by that command, just as if you'd type that output at that place in the command line.

🐧|🍏:
```bash
ls | cat -n | while read n f; do mv "$f" `printf "%03d.extension" $n`; done
```

You can also use **$()** to nest expressions: `mv "$f" $(printf "%03d.extension" $n)`.

<details>
  <summary><b>$f</b> instead <b>"$f"</b>?</summary>
  <b>$f</b> instead of <b>"$f"</b> fails when filename contains spaces!
  The main difference is that the quoted version is not subject to field splitting by the shell.
  With double quotes the outcome of the command expansion would be fed as one parameter to the source command. 
  Without quotes it would be broken up into multiple parameters, depending on the value of IFS (internal field separator) which contains space, TAB and newline by default.
  If the directory name does not contain such spaces then field splitting does not occur.
  As a rule of thumb, it is best to use double quotes with command substitutions and variable expansions.
</details>

## Find files

Find files modified on a specific date range.

🐧|🍏:
```bash
find . -type f -newermt "2022-12-01" ! -newermt "2023-01-01"
```

Find ASCII files and extract IP addresses.

🐧|🍏:
```bash
find . -type f -exec grep -Iq . {} \; -exec grep -oE "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)" {} /dev/null \;
```

## Enter the matrix

🐧|🍏:
```bash
perl -e '$|++; while (1) { print " ." x (rand(10) + 1), int(rand(2)) }'
```
