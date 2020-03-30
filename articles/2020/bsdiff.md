### BSDiff更新差量算法

在mac上安装环境相对简单，[官网](http://www.daemonology.net/bsdiff/)下载文件，解压后得到一个文件夹如下：

```bash
bsdiff-4.3
  +--bsdiff.1
  +--bsdiff.c
  +--bspatch.1
  +--bspatch.c
  +--Makefile
```

**修改Makefile**，在`.ifndef WITHOUT_MAN`与`.endif`前面加上一个tab缩进，如下：

```c
CFLAGS        +=    -O3 -lbz2

PREFIX        ?=    /usr/local
INSTALL_PROGRAM    ?=    ${INSTALL} -c -s -m 555
INSTALL_MAN    ?=    ${INSTALL} -c -m 444

all:        bsdiff bspatch
bsdiff:        bsdiff.c
bspatch:    bspatch.c

install:
    ${INSTALL_PROGRAM} bsdiff bspatch ${PREFIX}/bin
    .ifndef WITHOUT_MAN
    ${INSTALL_MAN} bsdiff.1 bspatch.1 ${PREFIX}/man/man1
    .endif
```

**修改bspatch.c**，加入#include <sys/types.h>

执行

```bash
$ make
```

执行成功后生成文件`bsdiff`与`bspatch`

生成两个文件的差分文件：

```bash
$ ./bsdiff old_file new_file diff_file
```

根据旧文件与差分文件生成新文件

```bash
$ ./bspatch old_file new_file diff_file
```
