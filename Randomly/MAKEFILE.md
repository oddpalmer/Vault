# Makefile 完整速查手册

> 基于上下文总结，支持快速查找和复习

## 目录
- [基础语法](#基础语法)
- [变量](#变量)
- [自动变量](#自动变量)
- [模式规则](#模式规则)
- [函数](#函数)
- [自动依赖生成](#自动依赖生成)
- [完整模板](#完整模板)
- [常见错误](#常见错误)
- [命令速查](#命令速查)

---

## 基础语法

### 基本格式
```makefile
target: dependencies
	command          # 注意：必须是 Tab 键，不能用空格
```

### 伪目标
```makefile
.PHONY: clean all    # 避免和同名文件冲突

clean:
	rm -f *.o
```

### 注释
```makefile
# 这是注释
CC = gcc  # 行尾注释也可以
```

---

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

| 变量 | 含义 | 示例（编译 test.o 时） |
|------|------|----------------------|
| `$@` | 当前目标的名称 | `test.o` |
| `$<` | 第一个依赖的文件名 | `test.c` |
| `$^` | 所有依赖的文件名（去重） | `test.c` |
| `$?` | 所有比目标新的依赖 | `test.c` |
| `$*` | 目标的主干名称（不含扩展名） | `test` |

### 使用示例
```makefile
# 编译规则
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

## 自动依赖生成

### 方法1：使用 -MMD 标志（推荐）
```makefile
CFLAGS += -MMD  # 自动生成 .d 依赖文件

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 包含依赖文件
DEPS = $(OBJS:.o=.d)
-include $(DEPS)

clean:
	rm -f $(OBJS) $(TARGET) *.d
```

### 方法2：手动生成依赖
```makefile
%.d: %.c
	@$(CC) -MM $(CFLAGS) $< > $@
	@sed -i 's|$(*F)\.o|$*.o|' $@

-include $(DEPS)
```

### -MMD 说明
```makefile
-MMD     # 生成依赖文件（不包含系统头文件）
-MD      # 生成依赖文件（包含系统头文件）
-MM      # 只输出依赖关系（不生成文件）
```

---

## 完整模板

### 模板1：小型项目（单目录）
```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = myapp

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: clean
```

### 模板2：中型项目（带头文件目录）
```makefile
CC = gcc
CFLAGS = -Wall -g -Iinclude
LDFLAGS = 
TARGET = myapp

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)

# 自动生成依赖
DEPENDS = $(OBJS:.o=.d)
CFLAGS += -MMD

$(TARGET): $(OBJS)
	$(CC) $^ -o $@ $(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

-include $(DEPENDS)

clean:
	rm -f $(OBJS) $(TARGET) *.d

run: $(TARGET)
	./$(TARGET)

.PHONY: clean run
```

### 模板3：大型项目（多目录结构）
```makefile
CC = gcc
CFLAGS = -Wall -g -Iinclude
LDFLAGS = 

SRC_DIR = src
OBJ_DIR = obj
BIN_DIR = bin
TARGET = $(BIN_DIR)/app

# 查找所有 .c
SRCS = $(shell find $(SRC_DIR) -name "*.c")
OBJS = $(patsubst $(SRC_DIR)/%.c, $(OBJ_DIR)/%.o, $(SRCS))

# 创建目录
$(shell mkdir -p $(OBJ_DIR) $(BIN_DIR))

# 依赖文件
DEPENDS = $(OBJS:.o=.d)
CFLAGS += -MMD

# 链接
$(TARGET): $(OBJS)
	$(CC) $^ -o $@ $(LDFLAGS)

# 编译
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $< -o $@

# 包含依赖
-include $(DEPENDS)

# 清理
clean:
	rm -rf $(OBJ_DIR) $(BIN_DIR)

# 重建
rebuild: clean $(TARGET)

.PHONY: clean rebuild
```

---

## 常见错误

| 错误信息                                    | 原因         | 解决方法            |
| --------------------------------------- | ---------- | --------------- |
| `missing separator`                     | 用了空格而非 Tab | 改为 Tab 键        |
| `No rule to make target`                | 依赖文件不存在    | 检查文件名和路径        |
| `commands commence before first target` | 命令前有空行     | 删除空行            |
| `multiple target patterns`              | 模式规则写错了    | 检查 `%` 的使用      |
| `missing separator`                     | Tab 被转成空格  | 编辑器设置 Tab 为 Tab |

### 变量替换错误
```makefile
# 错误（有空格）
OBJS = $(SRCS: .c = .o)

# 正确（无空格）
OBJS = $(SRCS:.c=.o)
```

### 链接命令错误
```makefile
# 错误
gcc test -o test.o myprint.o

# 正确
gcc test.o myprint.o -o test
# 语法：gcc 输入文件 -o 输出文件
```

### 编译命令错误
```makefile
# 错误（缺少 -c）
gcc test.c -o test.o

# 正确
gcc -c test.c -o test.o
```

---

## 命令速查

### 常用 make 命令
```bash
make           # 编译第一个目标
make clean     # 清理
make -j4       # 4 线程并行编译
make -n        # 预览命令（不执行）
make -B        # 强制重新编译
make -d        # 显示调试信息
make -p        # 打印所有规则和变量
make --trace   # 追踪执行过程
```

### 调试技巧
```bash
# 查看变量值
info:
	@echo "SRCS = $(SRCS)"
	@echo "OBJS = $(OBJS)"

# 或使用 make 内置
make -p | grep "SRCS"
```

### 编译选项
```bash
-Wall     # 显示所有警告
-g        # 包含调试信息
-O2       # 优化级别
-Ipath    # 添加头文件路径
-Lpath    # 添加库文件路径
-lname    # 链接库（如 -lm）
-c        # 只编译不链接
-o file   # 指定输出文件
-DMACRO   # 定义宏
-MMD      # 生成依赖关系
```

---

## 快速记忆口诀

1. **目标依赖第一行，Tab 缩进命令放**
2. **$@ 是目标，$< 第一个依赖，$^ 所有依赖**
3. **wildcard 找文件，patsubst 换后缀**
4. **.PHONY 声明伪目标，避免文件和它重**
5. **百分号 % 通配符，一条规则顶无数**
6. **编译用 -c，链接不要用 -c**

---

## 一键复制（最简模板）

```makefile
# 最简实用模板
CC = gcc
CFLAGS = -Wall -g -MMD
TARGET = myapp
SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

-include $(OBJS:.o=.d)

clean:
	rm -f $(OBJS) $(TARGET) *.d

.PHONY: clean
```

**使用：** 复制到项目目录，修改 `TARGET` 名称即可。添加新文件无需修改 Makefile！