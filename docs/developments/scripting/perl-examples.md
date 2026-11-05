# Perl Examples

## Table of Contents <!-- omit in toc -->

- [One-Liner Tips](#one-liner-tips)
  - [Display Matching Lines](#display-matching-lines)
  - [Delete Lines Containing Specific Characters](#delete-lines-containing-specific-characters)
  - [Comment Out Lines](#comment-out-lines)
- [Script Tips](#script-tips)
  - [Separate Filename and Extension](#separate-filename-and-extension)
  - [Skip Blank Lines](#skip-blank-lines)
  - [Trim Leading and Trailing Whitespace](#trim-leading-and-trailing-whitespace)
  - [Convert Lowercase to Uppercase](#convert-lowercase-to-uppercase)
  - [Report Filesystem Usage Exceeding Threshold](#report-filesystem-usage-exceeding-threshold)
  - [Check Process Execution Status](#check-process-execution-status)
  - [Generate Random Password](#generate-random-password)

## One-Liner Tips

### Display Matching Lines

```bash
perl -ne 'print /{expression}/' <file path>
```

```bash
perl -ne 'print unless /{expression}/' <file path>
```

### Delete Lines Containing Specific Characters

```bash
perl -ne 'print unless /{expression}/' <file path>
```

```bash
perl -ne 'print /{expression}/ or print' <file path>
```

### Comment Out Lines

```bash
perl -ne 'if ( /start/ .. /end/ ) { print "// $_" } else { print }'
```

## Script Tips

### Separate Filename and Extension

`separate.pl`:

```perl
#!/usr/bin/env perl
#
# ファイルと拡張子を分離する
#
use strict;
use warnings;

my $file_old = 'xxx.txt';
$file_old =~ /^(.+)(\.[^\.]+)$/;
my $basename = $1;
my $ext      = $2;  
my $file_new = "${basename}_edit.txt";
```

### Skip Blank Lines

`skip-blank-line.pl`:

```perl
#!/usr/bin/env perl
#
# 空行を読み飛ばして出力する
#
use strict;
use warnings;

while (my $line = <>){
    next if $line =~ /^\n/;
    print $line;
}
```

### Trim Leading and Trailing Whitespace

`trim.pl`:

```perl
#!/usr/bin/env perl
#
# 行の前後の空白を除去して出力: Trim
#
use strict;
use warnings;

while (my $line = <>) {
    $line =~ s/^\s+//;
    $line =~ s/\s+$//;
    print "$line\n";
}
```

### Convert Lowercase to Uppercase

`to-uppercase.pl`:

```perl
#!/usr/bin/env perl
#
# 小文字を大文字に変換する: Upper
#
use strict;
use warnings;

while (my $line = <>){
    $line =~ tr/a-z/A-Z/;
    print $line;
}
```

### Report Filesystem Usage Exceeding Threshold

`usage-filesystem.pl`:

<!-- spell-checker: disable -->
```perl
#!/usr/bin/env perl
#
# 指定した使用率を越えたファイルシステムを報告する
# が、dfの書式が違い使えない
#
$val = 80;
$msg = "";
open(DF, "/bin/df |");
while (<DF>) {
    chomp($_);
    @dfout = split(/[\t\s]+/, $_);
    $device = $dfout[0];
    $percent = $dfout[4];
    $mount = $dfout[5];
    if ($device =~ /\/dev/) {
        $msg .= "Filesystem:$device($mount) Use:$percent\n";
        print $msg;
        if ($percent > $val) {
            $msg .= "Filesystem:$device($mount) Use:$percent\n";
        }
    }
}
close(DF);
print $msg;
```
<!-- spell-checker: enable -->

### Check Process Execution Status

`check-process.pl`:

<!-- spell-checker: disable -->
```perl
#!/usr/bin/env perl
#
# プログラムが起動しているかどうかを確認する。
#
@pname = ('named', 'dhcpd', 'sshd', 'ntpd');
$msg = "";
foreach $p (@pname) {
    $alive = 0;
    open(PS, "/bin/ps ax | /bin/grep $p | grep -v grep | ");
    while (<PS>) {
        $alive = 1;
    }
    close(PS);
    if (!$alive) {
        $msg .= "$p: NOT ALIVE\n";
    }
}
print $msg;
```
<!-- spell-checker: enable -->

### Generate Random Password

`generate-password.pl`:

<!-- spell-checker: disable -->
```perl
#!/usr/bin/env perl
#
# ランダムなパスワードを生成
#
# 紛らわしい文字を抜いてもOK( 0o,!|1, など)
$passwd_char = '+-/\_^~;<>' . '23456789' . 'ABCDEFGHIJLMNPQRSTUVWXYZ' . 'abcdefghijklmnpqrstuvwxyz';
$len = 12;
$passwd = "";
$size = length($passwd_char);
for ($n = 0; $n < $len; $n++ ) {
    $passwd .= substr($passwd_char, int(rand($size)), 1);
}
print "$passwd\n";
```
<!-- spell-checker: enable -->
