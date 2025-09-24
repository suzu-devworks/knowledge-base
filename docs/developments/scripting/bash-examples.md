# Bash

## Table of Contents <!-- omit in toc -->

- [Generate Range Numbers](#generate-range-numbers)
- [Next action of find command](#next-action-of-find-command)
- [Rename File Extensions in Bulk](#rename-file-extensions-in-bulk)
- [Delete Emacs backup files in bulk](#delete-emacs-backup-files-in-bulk)
- [Convert a DOS text file to UNIX format](#convert-a-dos-text-file-to-unix-format)
- [Convert a UNIX text file to DOS format](#convert-a-unix-text-file-to-dos-format)
- [Calculate the Difference Between Two Specified times](#calculate-the-difference-between-two-specified-times)
- [Find Files Containing a Specified Keyword](#find-files-containing-a-specified-keyword)
- [Output the Full Path of a Specified File](#output-the-full-path-of-a-specified-file)
- [Split a File with a Sequential Filename](#split-a-file-with-a-sequential-filename)
- [Visualize Disk Usage per Directory](#visualize-disk-usage-per-directory)
- [Analyzing the output of the vmstat command](#analyzing-the-output-of-the-vmstat-command)
- [find and Spaces in Filenames](#find-and-spaces-in-filenames)
- [Remove Leading/Trailing Spaces from Filenames](#remove-leadingtrailing-spaces-from-filenames)

## Generate Range Numbers

You can use the brace expansion feature of Bash to generate a sequence of numbers. If this feature is not available, use the seq command.

Example with brace expansion:

```bash
#!/bin/bash
for i in {01..10}; do
  name="test$i"
  echo $name
done
```

Example with seq command:

```sh
#!/bin/sh
for i in $(seq 1 10); do
  name=`printf 'test%02d' $i`
  echo $name
done
```

## Next action of find command

Move all results

```sh
#!/bin/sh
find / -name *-nodate | xargs -I%_ mv %_ "<PATH>"
```
<!-- spell-checker: words nodate -->

## Rename File Extensions in Bulk

```sh
#!/bin/sh
for i in $1; do
  x=0
  a=.$2
  while [ -f $i$a ]; do
    x=`expr $x + 1`
    a=.$2$x
  done
  mv $i $i$a
done
```

## Delete Emacs backup files in bulk

```bash
#!/bin/bash
for fname in $(find . -name '*[~#]' -print)
do
  echo "${fname}"
  rm "${fname}"
done
```
<!-- spell-checker: words fname -->

## Convert a DOS text file to UNIX format

```bash
#!/bin/bash
infile="$1"
outfile="$2"
cat "${infile}" | tr -d '\015' | iconv -f CP932 -t UTF8 > "${outfile}"
```
<!-- spell-checker: words infile -->

## Convert a UNIX text file to DOS format

```bash
#!/bin/bash
infile="$1"
outfile="$2"
cat "${infile}" | sed "s/$/`echo -ne '\015'`/" | iconv -f UTF8 -t CP932 > "${outfile}"
```

## Calculate the Difference Between Two Specified times

```bash
#!/bin/sh
case $# in
  1) after=`date "+%s"` ;;
  2) after=`date "+%s" -d "$2"` ;;
  *) echo "usage: $0 before [after]"
    exit 1
    ;;
esac
before=`date "+%s" -d "$1"`
echo $after $before | awk '
  END {
    diff=$1-$2
    if (diff<0) diff=-diff
    printf("%02d:%02d:%02d\n", diff/3600, diff%3600/60, diff%3600%60)
  }
'
```
<!-- spell-checker: words esac -->

## Find Files Containing a Specified Keyword

```bash
#!/bin/bash
fname="$1"
dir="$2"
if [ -n "${dir}" ]
then
  find "${dir}" -name "*${fname}*" -print
else
  find . -name "*${fname}*" -print
fi
```

## Output the Full Path of a Specified File

```sh
#!/bin/sh
for path; do
  case "$path" in
    /* )
      echo "$path" ;;
    . | .. )
      ( cd "$path"; echo `pwd`) ;;
    * )
      echo `pwd`/"$path" ;;
  esac
done
```

## Split a File with a Sequential Filename

```sh
#!/bin/sh
# e.g.
#  split.sh -b 10240 some.dat
#
for arg in "$@"; do
  args="$args $arg"
  last=$arg
done
split $args || { echo "split error"; exit; }
cnt=0
for file in x??; do
  mv $file $last.`expr 000$cnt : '.*\(...\)'`
  cnt=`expr $cnt + 1`
done
```

## Visualize Disk Usage per Directory

```sh
#!/bin/sh
find "$1" \-type d \-maxdepth 1 \-mindepth 1 | xargs du \-sk | awk '
  {
    total \+= $1
    pathes\[$2\] \= $1
  }
  END {
    size \= 20
    for (name in pathes) {
      ratio \= pathes\[name\] / total
      cnt \= int(size \* ratio)
      printf("%6.3f%% ", ratio\*100)
      for (i \= 0; i \< cnt; i++) printf("#")
      for (i \= cnt; i \< size; i++) printf("-")
      printf(" %s\\n", name)
    }
  }' | sort \-nr
```
<!-- spell-checker: words pathes -->

## Analyzing the output of the vmstat command

```sh
#!/bin/sh
vm_file="log$$"
vm_path="/tmp/$vm_file"
egrep -v 'procs|swpd' $1 | sed -n -e '2,$ p' > $vm_path
( echo load\(\"$vm_path\"\)
  echo r = statistics\($vm_file\)
  echo save -ascii result r ) | octave -q > /dev/null
tail -9 result > result.log
rm $vm_path result
```
<!-- spell-checker: words procs swpd -->

## find and Spaces in Filenames

To prevent issues with spaces in directory names when looping with for, you can change the Internal Field Separator (IFS).

```sh
#!/bin/sh
IFS='\n'
for p in $(find . -iname '*.mp3' -print); do
  echo "$p"
done
```

## Remove Leading/Trailing Spaces from Filenames

```sh
#!/bin/sh
match_exec () {
  old_name=$1
  case $old_name in
    \ * )
      ;&
    *\ )
      new_name=`echo $old_name | sed -e 's/^[ ]*//' -e 's/[ ]*$//'`
      echo "[$old_name]"
      mv "$old_name" "$new_name"
      ;;
  esac
}

for item in *; do
  if [ -f "$item" ]; then
    match_exec "$item"
  elif [ -d "$item" ]; then
    (cd "$item"; $0;)
    match_exec "$item"
  fi
done
```
<!-- spell-checker: words elif -->
