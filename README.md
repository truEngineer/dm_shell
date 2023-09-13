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

<details>
  <summary>Argument list too long</summary>
  
  #### Colab Issue
  
  It's a kernel limitation on the size of the command line argument. This is a system issue, related to `execve` and `ARG_MAX` constant.
  Basically, the expansion produce a command (with its parameters) that exceeds the `ARG_MAX` limit.
  You can use a `for` loop instead, or the `find -exec` solution, which is much faster than a `for` loop.
  
  ```bash
  find . -name "*.jpeg" -print0 | xargs -0 md5sum | sort | awk 'BEGIN{lasthash = ""} $1 == lasthash {print $2} {lasthash = $1}' | xargs rm
  ```
  
  As noted above, the `for` loop approach is slower but more maintainable because it can adapt to more complex scenarios.
</details>

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
  
  **$f** instead of **"$f"** fails when filename contains spaces!
  
  The main difference is that the quoted version is not subject to field splitting by the shell.
  With double quotes the outcome of the command expansion would be fed as one parameter to the source command.
  Without quotes it would be broken up into multiple parameters, depending on the value of `IFS` (internal field separator) which contains space, `TAB` and newline by default.
  If the directory name does not contain such spaces then field splitting does not occur.
  
  As a rule of thumb, it is best to use double quotes with command substitutions and variable expansions.
</details>

## Finding specific files and data inside files

🐧|🍏:

Find files modified on a specific date range.

```bash
find . -type f -newermt "2022-12-01" ! -newermt "2023-01-01"
```

Find ASCII files and extract IP addresses.

```bash
find . -type f -exec grep -Iq . {} \; -exec grep -oE "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)" {} /dev/null \;
```

Search recursively to find a word or phrase in certain file types, such as C code.

```bash
find . -name "*.[ch]" -exec  grep -i -H "search phrase" {} \;
```

Find files in directory containing text.

```bash
grep -lir "some text" ./directory/*
```

## Download Files

**Download Google Drive Files using `wget`**

Files smaller than 100 MB are considered small files, while files larger than 100 MB are considered large.

Copy the link to share the file `https://drive.google.com/file/d/1UibyVC_C2hoT_XEw15gPEwPW4yFyJFeOEA/view?usp=sharing` (anyone who has a link can view).

Extract `FILEID` part `1UibyVC_C2hoT_XEw15gPEwPW4yFyJFeOEA`.

For a small file, run the following command on your terminal.

🐧|🍏:
```bash
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O FILENAME
```

For a large file, run the following command, making the necessary changes to `FILEID` and `FILENAME`.

🐧|🍏:
```bash
wget --load-cookies /tmp/cookies.txt \
"https://docs.google.com/uc?export=download&confirm=\
$(wget --quiet --save-cookies /tmp/cookies.txt \
--keep-session-cookies --no-check-certificate \
'https://docs.google.com/uc?export=download&id=FILEID' \
-O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=FILEID" \
-O FILENAME && rm -rf /tmp/cookies.txt
```

**Download Google Drive Files using `gdown`**

Download a large file from Google Drive.
If you use `curl/wget`, it fails with a large file because of the security warning from Google Drive. 
[Gdown](https://github.com/wkentaro/gdown) supports downloading from Google Drive folders (max 50 files per folder).

Installation.

```bash
$ pip install gdown
```

Usage from command line.

```bash
$ gdown --help
usage: gdown [-h] [-V] [-O OUTPUT] [-q] [--fuzzy] [--id] [--proxy PROXY]
             [--speed SPEED] [--no-cookies] [--no-check-certificate]
             [--continue] [--folder] [--remaining-ok]
             url_or_id

$ # a large file (~500MB)
$ gdown https://drive.google.com/uc?id=1l_5RK28JRL19wpT22B-DY9We3TVXnnQQ
$ md5sum fcn8s_from_caffe.npz
256c2a8235c1c65e62e48d3284fbd384

$ # same as the above but with the file ID
$ gdown 1l_5RK28JRL19wpT22B-DY9We3TVXnnQQ

$ # a small file
$ gdown https://drive.google.com/uc?id=0B9P1L--7Wd2vU3VUVlFnbTgtS2c
$ cat spam.txt
spam

$ # download with fuzzy extraction of a file ID
$ gdown --fuzzy 'https://drive.google.com/file/d/0B9P1L--7Wd2vU3VUVlFnbTgtS2c/view?usp=sharing&resourcekey=0-WWs_XOSctfaY_0-sJBKRSQ'
$ cat spam.txt
spam

$ # a folder
$ gdown https://drive.google.com/drive/folders/15uNXeRBIhVvZJIhL4yTw4IsStMhUaaxl -O /tmp/folder --folder
```

## Process CSV files

Print the first column of a CSV file.

🐧|🍏:
```bash
awk -F, '{print $1}' file.csv
```

Print the first and third columns of a CSV file.

🐧|🍏:
```bash
awk -F, '{print $1 "," $3}' file.csv
```

Print only the lines of a CSV file that contain a specific string.

🐧|🍏:
```bash
grep "string_of_interest" file.csv
```

Sort a CSV file based on the values in the second column.

🐧|🍏:
```bash
sort -t, -k2 file.csv
```
Remove the first row of a CSV file (the header row).

🐧|🍏:
```bash
tail -n +2 file.csv
```

Remove duplicates from a CSV file based on the values in the first column.

🐧|🍏:
```bash
awk -F, '!seen[$1]++' file.csv
```

Calculate the sum of the values in the third column of a CSV file.

🐧|🍏:
```bash
awk -F, '{sum+=$3} END {print sum}' file.csv
```

Convert a CSV file to a JSON array.

🐧|🍏:
```bash
jq -R -r 'split(",") | {name:.[0],age:.[1]}' file.csv
```

Convert a CSV file to a SQL INSERT statement.

🐧|🍏:
```bash
awk -F, '{printf "INSERT INTO table VALUES (\"%s\", \"%s\", \"%s\");\n", $1, $2, $3}' file.csv
```

## Enter the matrix

🐧|🍏:
```bash
perl -e '$|++; while (1) { print " ." x (rand(10) + 1), int(rand(2)) }'
```
