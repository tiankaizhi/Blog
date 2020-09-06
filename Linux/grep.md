## grep

```grep```（```Global``` search ```Regular``` ```Expression``` and ```Print``` out the line），Global search 意思是全局搜索，Regular Expression 表示正则表达式，Print out the line 意思是输出行，合起来就是按照正则表达式进行全局搜索并且输出对应的行。Unix 的 ```grep``` 家族包括 ```grep```、```egrep``` 和 ```fgrep```，Windows 系统下类似命令 ```FINDSTR```。

```
grep --help
```

```
[root@centos7 /]# grep --help
用法: grep [选项]... PATTERN [FILE]...
在每个 FILE 或是标准输入中查找 PATTERN。
默认的 PATTERN 是一个基本正则表达式(缩写为 BRE)。
例如: grep -i 'hello world' menu.h main.c

正则表达式选择与解释:
  -E, --extended-regexp     PATTERN 是一个可扩展的正则表达式(缩写为 ERE)
  -F, --fixed-strings       PATTERN 是一组由断行符分隔的定长字符串。
  -G, --basic-regexp        PATTERN 是一个基本正则表达式(缩写为 BRE)
  -P, --perl-regexp         PATTERN 是一个 Perl 正则表达式
  -e, --regexp=PATTERN      用 PATTERN 来进行匹配操作
  -f, --file=FILE           从 FILE 中取得 PATTERN
  -i, --ignore-case         忽略大小写
  -w, --word-regexp         强制 PATTERN 仅完全匹配字词
  -x, --line-regexp         强制 PATTERN 仅完全匹配一行
  -z, --null-data           一个 0 字节的数据行，但不是空行

Miscellaneous:
  -s, --no-messages         suppress error messages
  -v, --invert-match        select non-matching lines
  -V, --version             display version information and exit
      --help                display this help text and exit

输出控制:
  -m, --max-count=NUM       NUM 次匹配后停止
  -b, --byte-offset         输出的同时打印字节偏移
  -n, --line-number         输出的同时打印行号
      --line-buffered       每行输出清空
  -H, --with-filename       为每一匹配项打印文件名
  -h, --no-filename         输出时不显示文件名前缀
      --label=LABEL         将LABEL 作为标准输入文件名前缀
  -o, --only-matching       show only the part of a line matching PATTERN
  -q, --quiet, --silent     suppress all normal output
      --binary-files=TYPE   assume that binary files are TYPE;
                            TYPE is 'binary', 'text', or 'without-match'
  -a, --text                equivalent to --binary-files=text
  -I                        equivalent to --binary-files=without-match
  -d, --directories=ACTION  how to handle directories;
                            ACTION is 'read', 'recurse', or 'skip'
  -D, --devices=ACTION      how to handle devices, FIFOs and sockets;
                            ACTION is 'read' or 'skip'
  -r, --recursive           like --directories=recurse
  -R, --dereference-recursive
                            likewise, but follow all symlinks
      --include=FILE_PATTERN
                            search only files that match FILE_PATTERN
      --exclude=FILE_PATTERN
                            skip files and directories matching FILE_PATTERN
      --exclude-from=FILE   skip files matching any file pattern from FILE
      --exclude-dir=PATTERN directories that match PATTERN will be skipped.
  -L, --files-without-match print only names of FILEs containing no match
  -l, --files-with-matches  print only names of FILEs containing matches
  -c, --count               print only a count of matching lines per FILE
  -T, --initial-tab         make tabs line up (if needed)
  -Z, --null                print 0 byte after FILE name

文件控制:
  -B, --before-context=NUM  打印以文本起始的NUM 行
  -A, --after-context=NUM   打印以文本结尾的NUM 行
  -C, --context=NUM         打印输出文本NUM 行
  -NUM                      same as --context=NUM
      --group-separator=SEP use SEP as a group separator
      --no-group-separator  use empty string as a group separator
      --color[=WHEN],
      --colour[=WHEN]       use markers to highlight the matching strings;
                            WHEN is 'always', 'never', or 'auto'
  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)
  -u, --unix-byte-offsets   report offsets as if CRs were not there
                            (MSDOS/Windows)

‘egrep’即‘grep -E’。‘fgrep’即‘grep -F’。
直接使用‘egrep’或是‘fgrep’均已不可行了。
若FILE 为 -，将读取标准输入。不带FILE，读取当前目录，除非命令行中指定了-r 选项。
如果少于两个FILE 参数，就要默认使用-h 参数。
如果有任意行被匹配，那退出状态为 0，否则为 1；
如果有错误产生，且未指定 -q 参数，那退出状态为 2。

请将错误报告给: bug-grep@gnu.org
GNU Grep 主页: <http://www.gnu.org/software/grep/>
GNU 软件的通用帮助: <http://www.gnu.org/gethelp/>

```

1、在文件中查找包含字符串 "a" 的行

```
grep "a" filename
grep "a" filename1 filename2 filename3
```

2、在文件中查找不包含字符串 "a" 的行

```
grep -v "a" filename
grep -v "a" filename1 filename2 filename3
```

3、忽略大小写

```
grep -i "a" filename
grep -y "a" filename
```

4、只打印出匹配到的字符串

```
grep -o "a" filename
```

5、统计文件中有多少行匹配的字符串

```
grep -e "a" -e "b" filename
```

6、递归查找，用于多级目录中的文件

```
grep -r "a" 在当前目录下进行查找
```

7、输出匹配的字符串及对应的行号

```
grep -n "a" filename
```

8、输出匹配的字符串的行以及之前的行

```
grep -B 10 "a" filename   输出 "a" 之前的 10 行
```

9、输出匹配的字符串的行以及之后的行

```
grep -A 10 "a" filename  输出 "a" 之后的 10 行
```

10、输出匹配的字符串的行以及之前的 10 行，之后的 10 行

```
grep -B 10 -A 10 "a" filename  输出 "a" 之前的 10 行和之后的 10 行
```

或者

```
grep -C 10 "a" filename  输出 "a" 之前的 10 行和之后的 10 行
```

11、输出在特定文件下匹配的字符串

```
grep "a" *.log  输出以文件名以 log 结尾的匹配字符串

grep "a" file*  输出以文件名以 file 开头的匹配字符串
```
