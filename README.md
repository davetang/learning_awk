# Learning AWK
2023-08-29

- [Motivation](#motivation)
- [Using AWK](#using-awk)
- [Getting started](#getting-started)
  - [Pre-defined variables](#pre-defined-variables)
  - [Syntax](#syntax)
  - [Examples](#examples)
- [Further reading](#further-reading)
- [Version](#version)

![Build
README](https://github.com/davetang/learning_awk/actions/workflows/create_readme.yml/badge.svg)

Learning AWK.

## Motivation

After failing to use AWK, I always resort back to using `perl -lane`.
This works fine but perhaps more people are familiar with AWK than Perl.
Here I’ll try to finally learn the AWK syntax.

## Using AWK

99% of the time I use AWK is because it can print specific columns even
with inconsistent delimiting. Consider the following output.

``` bash
for f in $(ls *.sh); do wc ${f}; done
```

     19  61 421 git_setup.sh
      88  248 1919 render.sh

Normally I use `cut` to print out specific columns but this does not
work well when the output is irregular.

``` bash
for f in $(ls *.sh); do wc ${f}; done | cut -f3-4 -d' '
```

     61
    88 

AWK handles this nicely (somehow).

``` bash
for f in $(ls *.sh); do wc ${f}; done | awk '{print $3,$4}'
```

    421 git_setup.sh
    1919 render.sh

I also know how to use the Output Field Separator.

``` bash
for f in $(ls *.sh); do wc ${f}; done | awk 'OFS="," {print $3,$4}'
```

    421,git_setup.sh
    1919,render.sh

And that’s more or less about it.

## Getting started

Following this [Getting Started with AWK
guide](https://linuxhandbook.com/awk-command-tutorial/).

### Pre-defined variables

- `RS/ORS` - record separator/output record separator set to a newline
  character by default
- `NR` - number record, i.e. line number

``` bash
head -4 readme.qmd | awk '{print NR}'
```

    1
    2
    3
    4

- `FS/OFS` - field separator/output field separator set to white space
  as default
- `NF` - number fields

``` bash
for f in $(ls *.sh); do wc ${f}; done | awk '{print NF}'
```

    4
    4

### Syntax

AWK uses the following syntax: `pattern { action }`; if the pattern is
non-zero, i.e. true, the action block is executed. `{print}` is the
default action block, so we *could* exclude it in the example below but
it makes it less readable.

``` bash
awk '1 {print}' data/test.dat
```

    CREDITS,EXPDATE,USER,GROUPS
    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    52,01    dec   2018,sonia,team
    25,01    jan   2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12    jun   2018,öle,team:support
            


    17,05 apr 2019,abhishek,guest

### Examples

Skip header.

``` bash
awk 'NR>1' data/test.dat
```

    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    52,01    dec   2018,sonia,team
    25,01    jan   2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12    jun   2018,öle,team:support
            


    17,05 apr 2019,abhishek,guest

Print specific lines.

``` bash
awk 'NR>1 && NR<=4' data/test.dat
```

    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    52,01    dec   2018,sonia,team

Print lines with 1 or more fields.

``` bash
awk 'NF' data/test.dat
```

    CREDITS,EXPDATE,USER,GROUPS
    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    52,01    dec   2018,sonia,team
    25,01    jan   2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12    jun   2018,öle,team:support
    17,05 apr 2019,abhishek,guest

Print specific fields; `$0` is the entire record but is not used in the
example. No pattern is specified, so an action is carried out for every
record.

``` bash
awk '{print $1,$3}' FS=, OFS=, data/test.dat
```

    CREDITS,USER
    99,sylvain
    52,sonia
    52,sonia
    25,sonia
    10,sylvain
    8,öle
            ,
    ,
    ,
    17,abhishek

There are `BEGIN` blocks in AWK, just like in Perl.

``` bash
awk 'BEGIN { FS=OFS="," } NF { print $1, $3 }' data/test.dat
```

    CREDITS,USER
    99,sylvain
    52,sonia
    52,sonia
    25,sonia
    10,sylvain
    8,öle
            ,
    17,abhishek

And also `END` blocks.

``` bash
awk '{ SUM+=$1 } END { print SUM }' FS=, OFS=, data/test.dat
```

    263

Using a regex.

``` bash
awk '!/^$/' data/test.dat
```

    CREDITS,EXPDATE,USER,GROUPS
    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    52,01    dec   2018,sonia,team
    25,01    jan   2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12    jun   2018,öle,team:support
            
    17,05 apr 2019,abhishek,guest

**All arrays in AWK are associative arrays**, i.e. hashes or
dictionaries. This makes it easy to remove duplicate lines.

``` bash
awk '!l[$0]++' data/test.dat
```

    CREDITS,EXPDATE,USER,GROUPS
    99,01 jun 2018,sylvain,team:::admin
    52,01    dec   2018,sonia,team
    25,01    jan   2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12    jun   2018,öle,team:support
            

    17,05 apr 2019,abhishek,guest

I don’t understand how this works, but the following removes multiple
spaces (default field separator).

``` bash
awk '$1=$1' data/test.dat
```

    CREDITS,EXPDATE,USER,GROUPS
    99,01 jun 2018,sylvain,team:::admin
    52,01 dec 2018,sonia,team
    52,01 dec 2018,sonia,team
    25,01 jan 2019,sonia,team
    10,01 jan 2019,sylvain,team:::admin
    8,12 jun 2018,öle,team:support
    17,05 apr 2019,abhishek,guest

`printf` where `%s` for strings, `%d` for integers, and `%f` for
floating points. The `+$1` forces the evaluation of `$1` in a numerical
context and therefore any record in column one that is not a number
returns 0 and is thus skipped. While it is a nice shortcut, it’s hard to
read.

``` bash
awk '+$1 { printf("%s ",  $3) }' FS=, data/test.dat
```

    sylvain sonia sonia sonia sylvain öle abhishek 

To upper case.

``` bash
awk '$3 { print toupper($0) }' data/test.dat
```

    99,01 JUN 2018,SYLVAIN,TEAM:::ADMIN
    52,01    DEC   2018,SONIA,TEAM
    52,01    DEC   2018,SONIA,TEAM
    25,01    JAN   2019,SONIA,TEAM
    10,01 JAN 2019,SYLVAIN,TEAM:::ADMIN
    8,12    JUN   2018,öLE,TEAM:SUPPORT
    17,05 APR 2019,ABHISHEK,GUEST

Search and replace with `gsub`.

``` bash
awk '+$1 { gsub(/ +/, "-", $2); print }' FS=, data/test.dat
```

    99 01-jun-2018 sylvain team:::admin
    52 01-dec-2018 sonia team
    52 01-dec-2018 sonia team
    25 01-jan-2019 sonia team
    10 01-jan-2019 sylvain team:::admin
    8 12-jun-2018 öle team:support
    17 05-apr-2019 abhishek guest

## Further reading

- [The GNU Awk User’s
  Guide](https://www.gnu.org/software/gawk/manual/gawk.html)

## Version

Bash version used to generate this document.

``` bash
bash --version
```

    GNU bash, version 5.1.16(1)-release (x86_64-pc-linux-gnu)
    Copyright (C) 2020 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

    This is free software; you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
