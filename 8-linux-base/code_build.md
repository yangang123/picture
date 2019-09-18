@[toc]

# C文件
> 新建hello.c，文件内容如下
```c
#include <stdio.h>

int main(int argc, char **argv)
{
	printf("hello world\n");
}
```

# 构建方法
-   Makefile
-   ninja 
-   cmake

# Makefile构建
> hello依赖hello.c，命令是gcc -o hello hello.c
```
hello:hello.c
	gcc -o hello hello.c
```

# ninja构建
> 需要
## rule.ninja文件内容
```
cflags = -Wall

rule cc
  command = gcc $cflags  $in -o $out
```

## build.ninja文件内容
```
include rule.ninja

build hello: cc hello.c

build all: phony hello

# 第一个目标
default all

```

# cmake构建
> cmake会把add_executable指定的目标在makefile中第一个目标，在build.ninja中通过“default all”指定第一个目标
```
project (hello)
add_executable(hello hello.c)
```

# 总结
make没有指定目标，会找到makefile文件的第一个目标作为目标输出， ninja会找build.ninja中的"default all" 中的“default”指定
的目标，如果"default"没有指定目标，才会找文件中的地一个目标

