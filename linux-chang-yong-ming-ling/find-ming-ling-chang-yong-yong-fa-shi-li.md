---
description: author:chi
---

# 🐧 find命令常用用法示例

## 最基础的打印操作

find命令默认接的命令是-print，它默认以\n将找到的文件分隔。可以使用-print0来使用\0分隔，这样就不会分行了。但是一定要注意，-print0针对的是\n转\0，如果查找的文件名本身就含有空格，则find后-print0仍然会显示空格文件。所以-print0实现的是\n转\0的标记，可以使用其他工具将\0标记替换掉，如xargs，tr等。

```sh
$ mkdir /tmp/a
$ touch /tmp/a/{1..5}.log
$ find /tmp/a   # 等价于find /tmp/a -print，表示搜索/tmp/a目录
/tmp/a
/tmp/a/4.log
/tmp/a/2.log
/tmp/a/5.log
/tmp/a/1.log
/tmp/a/3.log

$ find /tmp/a -print0    
/tmp/a/tmp/a/4.log/tmp/a/2.log/tmp/a/5.log/tmp/a/1.log/tmp/a/3.log
```

## 文件名搜索

常用的两个是-name和-path。

\-name可以对文件的basename进行匹配，-path可以对文件的dirname+basename。查找的文件名最好使用引号包围，可以配合通配符进行查找。

```bash
$ find /tmp -name "*.log"
/tmp/screen.log
/tmp/x.log
/tmp/timing.log
/tmp/a/4.log
/tmp/a/2.log
/tmp/a/5.log
/tmp/a/1.log
/tmp/a/3.log
/tmp/b.log
```

但不能在-name的模式中使用”/“，除非文件名中包含了字符”/“，否则将匹配不到任何东西，因为-name只对basename进行匹配。例如，想要匹配/tmp目录下某包含字符a的目录下的log文件

```bash
$ find /tmp -name '*a*/*.log'
find: warning: Unix filenames usually don't contain 
slashes (though pathnames do). That means that 
'-name ‘*a*/*.log’' will probably evaluate to false
all the time on this system.  You might find the 
'-wholename' test more useful, or perhaps '-samefile'.
Alternatively, if you are using GNU grep, you could
use 'find ... -print0 | grep -FzZ ‘*a*/*.log’'.
```

所以想要在指定目录下搜索某目录中的某文件，应该使用-path而不是-name。

```bash
$ find /tmp -path '*a*/*.log'
/tmp/abc/axyz.log
```

注意，配合通配符\[]时应该注意是基于字符顺序的，大小写字母的顺序是a-z –> A-Z，指定\[a-z]表示小写字母a-z，同理\[A-Z]，而\[a-zA-Z]和\[a-Z]都表示所有大小写字母。当然还可以指定\[a-A]表示a-z外加一个A。

字母的处理顺序较容易理解，关于数字的处理方法，见下面的示例。

```bash
$ ls
11.sh  1.sh  22.sh  2.sh  3.sh

$ find -name "[1-2].sh"
./2.sh
./1.sh

$ find -name "[1-23].sh"
./2.sh
./3.sh
./1.sh

$ touch 0.sh
$ find -name "[1-20].sh"
./2.sh
./0.sh
./1.sh

$ find -name "[1-22-3].sh"
./2.sh
./3.sh
./1.sh
```

从上面结果可以看出，其实\[]只能匹配单个字符，\[0-9]表示0-9的数字，\[1-20]表示\[1-2]外加一个0，\[1-23]表示\[1-2]外加一个3，\[1-22-3]表示\[1-2]或\[2-3]，迷惑点就是看上去是大于10的整数，其实是两个或者更多的单个数字组合体。也可以用这种方法表示多种匹配：\[1-2,2-3]。

## 根据文件类型搜索：-type

一般需要搜索的文件类型就只有普通文件(f)，目录(d)，链接文件(l)。

例如，搜索普通文件类的文件，且名称为a开头的sh文件。

```bash
$ find /tmp -type f -name "a*.sh"

搜索目录类文件，且目录名以a开头。

$ find /tmp -type d -name "a*"
```

## 根据文件的时间戳搜索

最基础的时间戳包括：-atime/-mtime/-ctime。

> atime(Access Time) -文件的最近访问时间(cat、vim等) mtime(Modify Time) -文件的内容最近修改的时间(vim编辑,重定向写入等) --- 重点检查对象 ctime(Change Time) -文件属性最近修改的时间(内容、路径、所有者、权限等变化)

例如搜索/tmp下3天内修改过内容的sh文件，因为是文件内容，所以不考虑搜索目录。

```bash
$ find /tmp -type f -mtime -3 -name "*.sh"
```

## 根据文件大小搜索：-size

例如搜索/tmp下大于100K的sh文件

```bash
$ find /tmp -type f -size +100k -name '*.sh'
```

## 根据权限搜索：-perm

例如搜索/tmp下所有者具有可读可写可执行权限的sh文件。

```bash
find /tmp -type f -perm -0700 -name '*.sh'
```

## 搜索空文件

空文件可以是没有任何内容的普通文件，也可以是没有任何内容的目录。

例如搜索目录中没有文件的空目录。

```bash
$ find /tmp -type d -empty
```

## 搜索到文件后并删除

例如搜索到/tmp下的”.tmp”文件然后删除。

```bash
$ find /tmp -type f -name "*.tmp" -exec rm -rf  '{}'  \;
```

## 搜索指定日期范围的文件

例如搜索/test下2021-06-03到2021-06-06之间修改过的文件。

```bash
$ find /test -type f -newermt 2021-06-03 -a ! -newermt 2021-06-06
```

或者，创建两个临时文件，并用touch修改这两个文件的修改时间，然后find -newer去参照这两个文件。

```bash
$ touch -m -d 2017-06-03 tmp1.txt
$ touch -m -d 2017-06-06 tmp2.txt
$ find /test -type f -newer tmp1.txt -a ! -newer tmp2.txt
```

## 并行加速搜索

有时候，想要搜索的内容并不知道在哪里，这时我们会从根”/“开始搜索，这样的搜索速度可能会稍微长那么一点点。为了加速搜索，使用xargs的并行功能。例如，搜索”/“下的所有”Find.pm”结尾的文件：

```bash
ls --hide proc / | xargs -i -P 0 find /{} -type f -name "*Find.pm"
```

## 获取文件绝对路径

当find结合管道，而管道后的命令很可能想要获取到搜索到的文件的绝对路径，或者说是全路径。而问题是，当find的搜索路径是相对路径时，搜索出来的显示结果也是以相对路径显示的。

```bash
$ mkdir /tmp/test
$ touch /tmp/test/{a,b,c}.png
$ find .
.
./a.png
./b.png
./c.png
```

想要获取全路径，方式有很多种：

```bash
# 搜索前先pwd

$ find $(pwd)
/tmp/test
/tmp/test/a.png
/tmp/test/b.png
/tmp/test/c.png

# 或使用$PWD环境变量
$ find $PWD
/tmp/test
/tmp/test/a.png
/tmp/test/b.png
/tmp/test/c.png

# 使用bash的波浪号扩展 `~+`
$ find ~+
/tmp/test
/tmp/test/a.png
/tmp/test/b.png
/tmp/test/c.png
```

## 获取文件名部分(basename)

find的-printf选项有很多修饰符功能，对于处理路径方面的修饰符有%f、%p、%P，其中%f是获取basename（去除所有路径前缀），%p是获取路径自身，一般用不上，%P是获取除了find搜索路径的剩余部分。

首先，想要获取basename，建议使用%f。

```bash
$ mkdir /tmp/test/test1
$ touch /tmp/test/test1/{x,y,z}.png
$ find /tmp/test -printf "%f\n"
test
a.png
b.png
c.png
test1
x.png
y.png
z.png
```

再看使用%P的效果。结果仅仅是去掉了find搜索路径/tmp/test部分。当搜索路径只有一层(即没有子目录)时，它也可以用来获取basename。

```bash
$ find /tmp/test -printf "%P\n"

a.png
b.png
c.png
test1
test1/x.png
test1/y.png
test1/z.png
```

## 从结果中排除目录自身

find搜索目录时，总是会将搜索路径自身也包含到搜索结果中。想办法排除它是必须的。

排除的方法是，加上一个-path选项并取反，-path的参数和find的搜索路径参数必须一致。

```bash
$ find /tmp/test ! -path /tmp/test
/tmp/test/a.png
/tmp/test/b.png
/tmp/test/c.png
/tmp/test/test1
/tmp/test/test1/x.png
/tmp/test/test1/y.png
/tmp/test/test1/z.png

$ find . ! -path .
./a.png
./b.png
./c.png
./test1
./test1/x.png
./test1/y.png
./test1/z.png
```
