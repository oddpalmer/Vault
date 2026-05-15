# Makefile

## 变量

### 定义与使用
```makefile
# 定义
CC = gcc
CFLAGS = -Wall -g
SRCS = main.c utils.c

# 使用 - 两种格式
$(CC) $(CFLAGS)
${CC} ${CFLAGS}
```

### 常用变量命名
```makefile
CC = gcc              # 编译器
CXX = g++             # C++编译器
CFLAGS = -Wall -g     # 编译选项
CXXFLAGS = -Wall -g   # C++编译选项
LDFLAGS = -lm         # 链接选项
TARGET = myapp        # 目标文件名
SRCS = main.c util.c  # 源文件
OBJS = main.o util.o  # 目标文件
```

### 变量赋值类型
```makefile
=    # 递归展开（使用时才展开）
:=   # 简单展开（立即展开）
?=   # 条件赋值（未定义时才赋值）
+=   # 追加
```

---

## 自动变量

| 变量   | 含义             | 示例（编译 test.o 时） |
| ---- | -------------- | --------------- |
| `$@` | 当前目标的名称        | `test.o`        |
| `$<` | 第一个依赖的文件名      | `test.c`        |
| `$^` | 所有依赖的文件名（去重）   | `test.c`        |
| `$?` | 所有比目标新的依赖      | `test.c`        |
| `$*` | 目标的主干名称（不含扩展名） | `test`          |
|      |                |                 |

### 使用示例
```makefile
# 编译规则 .o文件依赖.c文件
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
	# $< = test.c, $@ = test.o

# 链接规则
$(TARGET): $(OBJS)
	$(CC) $^ -o $@
	# $^ = test.o utils.o, $@ = myapp
```

---

## 模式规则

### 基本模式规则
```makefile
# % 是通配符，表示任意相同字符串
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

### 展开原理
```makefile
# 一条规则自动展开为：
# test.o: test.c
# utils.o: utils.c
# main.o: main.c
```

### 带目录的模式规则
```makefile
# 源文件在 src 目录，目标文件在 obj 目录
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $< -o $@
```

### 静态模式规则
```makefile
$(OBJS): %.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

---

## 函数

### 文件操作函数

| 函数 | 语法 | 示例 | 结果 |
|------|------|------|------|
| wildcard | `$(wildcard pattern)` | `$(wildcard *.c)` | `main.c utils.c` |
| patsubst | `$(patsubst pattern,replacement,text)` | `$(patsubst %.c,%.o,main.c)` | `main.o` |
| notdir | `$(notdir names)` | `$(notdir src/main.c)` | `main.c` |
| basename | `$(basename names)` | `$(basename main.c)` | `main` |
| dir | `$(dir names)` | `$(dir src/main.c)` | `src/` |

### 字符串替换简写
```makefile
# 替换后缀（常用）
OBJS = $(SRCS:.c=.o)      # 所有 .c 替换为 .o
OBJS = $(SRCS:%.c=%.o)    # 同上

# 去掉后缀
NAMES = $(basename $(SRCS))

# 添加前缀
OBJS = $(addprefix obj/, $(SRCS:.c=.o))
```

### 其他常用函数
```makefile
# 取第一个
FIRST = $(firstword $(SRCS))  # main.c

# 去重
UNIQ = $(sort $(LIST))

# 过滤
CFILES = $(filter %.c, $(SRCS))   # 只保留 .c
HFILES = $(filter %.h, $(SRCS))   # 只保留 .h

# shell 命令执行
SRCS = $(shell find src -name "*.c")
```

---



# GCC编译格式
```shell
gcc [选项] -o [输出文件] [输入文件] 
gcc [选项] [输入文件] -o [输出文件]
// 两种语法一致 
```

每个文件都有main入口，make自动化也能编译所有文件
```makefile
SRCS = $(wildcard *.c)
TARGETS = $(SRCS:.c=.out)

all: $(TARGETS)

%.out: %.c
	gcc -Wall -O2 -o $@ $<

clean:
	rm -f $(TARGETS)
```
