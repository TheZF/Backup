# CMake

## cmake一般流程

```cmake
project(xxx)                                          #必须

add_subdirectory(子文件夹名称)                         #父目录必须，子目录不必

add_library(库文件名称 STATIC 文件)                    #通常子目录(二选一)
add_executable(可执行文件名称 文件)                     #通常父目录(二选一)

include_directories(路径)                              #必须
link_directories(路径)                                 #必须

target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)       #必须
```



## 常用命令

```cmake
# 本CMakeLists.txt的project名称
# 会自动创建两个变量，PROJECT_SOURCE_DIR和PROJECT_NAME

# ${EXECUTABLE_OUTPUT_PATH}：目标二进制可执行文件的存放位置
# ${PROJECT_SOURCE_DIR}：本CMakeLists.txt所在的文件夹路径
# ${PROJECT_NAME}：本CMakeLists.txt的project名称
# ${LIBRARY_OUTPUT_PATH}：库文件的默认输出路径
project(xxx)

# 获取路径下所有的.cpp/.c/.cc文件，并赋值给变量中		路径.表示当前路径
aux_source_directory(路径 变量)
#缺点：会将路径下的所有源文件赋给变量，不够灵活

# 给文件名/路径名或其他字符串起别名，用${变量}获取变量内容，可自由选择源文件
set(变量 文件名/路径/...)

# 添加编译选项
add_definitions(编译选项)

# 打印消息
message(消息)

# 编译子文件夹的CMakeLists.txt
add_subdirectory(子文件夹名称)
##基本语法
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
#一共有三个参数，后两个是可选参数.
#source_dir 源代码目录
指定一个包含CMakeLists.txt和代码文件所在的目录，该目录可以是绝对路径，也可以是相对路径，对于后者相对路径的起点是CMAKE_CURRENT_SOURCE_DIR。此外，如果子目录再次包含的CMakeLists.txt，则将继续处理里层的CMakeLists.txt，而不是继续处理当前源代码。
#binary_dir 二进制代码目录
这个目录是可选的，如果指定，cmake命令执行后的输出文件将会存放在此处，若没有指定，默认情况等于source_dir没有进行相对路径计算前的路径，也就是CMAKE_BINARY_DIR。
#EXCLUDE_FROM_ALL标记
这个标志是可选的，如果传递了该参数表示新增加的子目录将会排除在ALL目录之外（可能是make系统中的make all？），表示这个目录将从IDE的工程中排除。用户必须显式在子文件这个编译目标（手动cmake之类的）。指定了这个文件夹，表示这个文件夹是独立于源工程的，这些函数是有用但是不是必要的，比如说我们一系列的例子。
EXCLUDE_FROM_ALL将会将这个目录从编译中排除，如工程的例子需要等待其他编译完成后再进行单独的编译。通常子目录应该包含自己的project()命令，这样以来整个编译命令将会产生各自的目标文件。如果把CMakeLists.txt与VS IDE比较，总的CMakeLists.txt就相当于解决方案，子CMakeLists.txt就相当于在解决方案下的工程文件。还有一个需要注意的是，如果编译父CMakeLists时依赖了子CMakeLists.txt中的源文件，那么该标志将会被覆盖（也就是也会处理），以满足编译任务。


# 将.cpp/.c/.cc文件生成.a静态库
# 注意，库文件名称通常为libxxx.so，在这里只要写xxx即可
add_library(库文件名称 STATIC 文件)
#生成动态库或静态库(第1个参数指定库的名字；第2个参数决定是动态还是静态，如果没有就默认静态；第3个参数指定生成库的源文件)

#设置最终生成的库名称，还可以设置其它属性，如路径
set_target_properties (库文件 PROPERTIES OUTPUT_NAME "生成的库名")

# 将.cpp/.c/.cc文件生成可执行文件
add_executable(可执行文件名称 文件)

# 添加搜索.h头文件的路径，多个路径可用空格分隔
include_directories(路径)

#在指定目录下查找指定库，并把库的绝对路径存放到变量里
find_library(变量 库名称 HINTS 路径)
#在执行cmake ..时就会去查找库是否存在，这样可以提前发现错误，不用等到链接时
#HINTS提供建议路径，PATHs强制指定路径。CMake搜索路径：PATHs，HINTS，系统默认路径

# 规定.so/.a库文件路径
link_directories(路径)

# 对add_library或add_executable生成的文件进行链接操作
# 注意，库文件名称通常为libxxx.so，在这里只要写xxx即可
target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)

#在目标构建完成后强制触发 CMake 的重新配置
add_custom_command(
    TARGET app                # 作用的目标（可执行文件或库）
    POST_BUILD               # 执行时机（在目标构建完成后运行） PRE_BUILD：构建开始前。PRE_LINK：链接开始前。
    COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_SOURCE_DIR}/CMakeLists.txt  # 执行的命令
    COMMENT "提示信息"  # 构建时的提示信息
)
#还可以在find_library()前添加unset (变量 CACTH)来清除缓存变量，确保每次配置重新搜索库
```

# Makefile

## 生成可执行文件模板

```makefile
# 定义变量
CC := g++
CFLAGS := -c  # 编译选项
LDFLAGS :=  # 链接选项
SRC_DIR := ./src    # 源代码目录
BIN_DIR := ./bin    # 输出目录
INCLUDE_DIR := ./include  # 头文件目录
LIB_DIR := ./lib    # 库文件目录

# 定义目标文件和源文件
TARGET := $(BIN_DIR)/main  # 可执行文件目标
OBJECTS := $(SRC_DIR)/main.o  # 中间目标文件
SOURCE := $(SRC_DIR)/main.cpp   # 源文件
HEADERS := $(INCLUDE_DIR)/math.h  # 头文件

# 默认目标
.PHONY: all
all: $(TARGET)

# 生成可执行文件，将动态库作为依赖项，动态库更新时可同步更新可执行文件
$(TARGET): $(OBJECTS) libXXX.so
	$(CC) $(LDFLAGS) $^ -L$(LIB_DIR) -lmath -o $@

# 编译源文件
$(OBJECTS): $(SOURCE) $(HEADERS)
	$(CC) $(CFLAGS) -I$(INCLUDE_DIR) $< -o $@

# 清理生成的文件
.PHONY: clean
clean:
	rm -f $(OBJECTS) $(TARGET)
```

## 生成静态库模板

```makefile
# 定义变量
CC := g++
CFLAGS := -c  # 编译选项
SRC_DIR := ./src    # 源代码目录
LIB_DIR := ./lib    # 输出目录
INCLUDE_DIR := ./include  # 头文件目录

# 定义目标文件和源文件
TARGET := $(LIB_DIR)/libmath.a  # 静态库目标文件
OBJECTS := $(SRC_DIR)/math.o    # 中间目标文件
SOURCE := $(SRC_DIR)/math.cpp   # 源文件
HEADERS := $(INCLUDE_DIR)/math.h  # 头文件

# 默认目标
.PHONY: all
all: $(TARGET)

# 生成静态库
$(TARGET): $(OBJECTS)
	ar rcs $@ $^

# 编译源文件
$(OBJECTS): $(SOURCE) $(HEADERS)
	$(CC) $(CFLAGS) -I$(INCLUDE_DIR) $< -o $@

# 清理生成的文件
.PHONY: clean
clean:
	rm -f $(OBJECTS) $(TARGET)
```

## 生成动态库通用模板

```makefile
# 定义变量
CC := g++
CFLAGS := -c -fPIC  # 编译选项，-fPIC 表示生成位置无关代码
LDFLAGS := -shared  # 链接选项，-shared 表示生成共享库
SRC_DIR := ./src    # 源代码目录
LIB_DIR := ./lib    # 输出目录
INCLUDE_DIR := ./include  # 头文件目录

# 定义目标文件和源文件
TARGET := $(LIB_DIR)/libmath.so  # 动态库目标文件
OBJECTS := $(SRC_DIR)/math.o    # 中间目标文件
SOURCE := $(SRC_DIR)/math.cpp   # 源文件
HEADERS := $(INCLUDE_DIR)/math.h  # 头文件

# 默认目标
.PHONY: all
all: $(TARGET)

# 生成动态库
$(TARGET): $(OBJECTS)
	$(CC) $(LDFLAGS) $^ -o $@

# 编译源文件
$(OBJECTS): $(SOURCE) $(HEADERS)
	$(CC) $(CFLAGS) -I$(INCLUDE_DIR) $< -o $@

# 清理生成的文件
.PHONY: clean
clean:
	rm -f $(OBJECTS) $(TARGET)
```

