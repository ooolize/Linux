### Makefile是什么
### 增量编译
将修改过的.c文件重新编译为.o文件在与目标文件链接
### 例子
+ 需要用tab严格对齐
目标项:依赖项
```
main:main.o func.o
    gcc main.o func.o -o main
main.o:main.c
    gcc -c main.c
func.o:func.c
    gcc -c func.c
```
+ 加上伪命令：clean
make clean清理文件
```
main:main.o func.o
    gcc main.o func.o -o main
main.o:main.c
    gcc -c main.c
func.o:func.c
    gcc -c func.c
.PHONY:clean
clean:
    rm -rf func.o main.o
```
加上用户自定义变量变量 

```
OBJS:=main.o func.o
ELF:=main
#(ELF):$(OBJS)
    gcc $(OBJS) -o ELF
 main.o:main.c
    gcc -c main.c
 func.o:func.c
    gcc -c func.c
.PHONY:clean
clean:
    rm -rf $(OBJS)
```
使用镜变量
$@ 当前文件的目标文件
$^ 当前文件的所有依赖文件
```
OBJS:=main.o func.o
ELF:=main
#(ELF):$(OBJS)
    gcc $^ -o $@
main.o:main.c
    gcc -c $^
func.o:func.c
    gcc -c $^
.PHONY:clean
clean:
    rm -rf $(OBJS)
```
加上预定义变量修改
```
OBJS:=main.o func.o
ELF:=main
CC:=gcc //预定义变量
CFALGS:=-g
#(ELF):$(OBJS)
    gcc $^ -o $@
.PHONY:clean
clean:
    rm -rf $(OBJS)
```
加上模式规则
SOURCES:E=$(wildcard,*c)//把当前文件所有c文件放到SOURCES

OBJS:=$(patsubst %.c,%.o,$(SOURCES))
```
SOURCES:E=$(wildcard,*c)
OBJS:=$(patsubst %.c,%.o,$(SOURCES))
ELF:=main
CC:=gcc //预定义变量
CFALGS:=-g
#(ELF):$(OBJS)
    gcc $^ -o $@
.PHONY:clean
clean:
    rm -rf $(OBJS)
```


