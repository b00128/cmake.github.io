# 

## 0 什么是CMake？

CMake 是一个跨平台的构建系统生成器。你可以把它想象成一个“构建工具的构建工具”。它本身并不直接编译代码，而是根据你提供的配置文件（通常是 `CMakeLists.txt`）来生成特定平台的构建文件（比如 Makefile、Visual Studio 项目文件等）。



## 1 CMake的工作流程

- **编写 `CMakeLists.txt`**

  这是 CMake 的配置文件，告诉 CMake 项目结构、源文件、依赖等信息。

- **运行 CMake**

  CMake 读取 `CMakeLists.txt`，生成平台特定的构建文件。

- **编译项目**

  使用生成的构建文件（如 Makefile）来编译项目。



## 2 基本语法

#### cmake_minimum_required()

```cmake
# 指定 CMake 的版本要求
cmake_minimum_required(VERSION <min>[...<max>] [FATAL_ERROR])
```

- 参数说明

  - `min` 项目要求的最低 CMake 版本
  - `max` 项目要求的最高 CMake 版本
  - `FATAL_ERROR` 若存在，当版本不满足时立即终止配置；若省略，仅显示警告（CMake 3.12+ 默认为错误）

- 示例

  ```cmake
  cmake_minimum_required(VERSION 3.10) # 要求 CMake ≥3.10
  cmake_minimum_required(VERSION 3.10...3.18) # 要求 3.10 ≤ CMake ≤ 3.18
  cmake_minimum_required(VERSION 3.10 FATAL_ERROR) # 若 CMake 版本低于 3.10，立即报错退出
  ```

---

### function

```cmake
function(<name> [<arg1> ...])
  <commands>
endfunction()
```

定义一个名为 `<name>` 的函数，该函数接受名为 `<arg1>`, ... 的参数。

#### 调用



### macro

```cmake
macro(<name> [<arg1> ...])
  <commands>
endmacro()
```

定义一个名为 `<name>` 的宏，它接受名为 `<arg1>`, ... 的参数。列在 macro 之后但在匹配的 [`endmacro()`](https://cmake.com.cn/cmake/help/latest/command/endmacro.html#command:endmacro) 之前的命令不会执行，直到宏被调用。



---

#### project()

##### 基础

```cmake
# 指定 CMake 项目名称、编程语言
project(<PROJECT-NAME> [<language-name>...])
```

- 参数说明	

  - `PROJECT_NAME` 项目名称
  - `language-name` 编程语言，默认为C/CXX

- 示例

  ```cmake
  # 示例
  project(MyLib) # 指定项目名称为MyLib
  project(MyLib C CXX) # 指定项目名称为MyLib，编程语言为C/C++
  ```

##### 拓展

```cmake
# 指定 CMake 项目名称，此外还可以指定版本号、描述、主页链接以及编程语言
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

- 参数说明	

  - `PROJECT-NAME` 项目名称
  - `VERSION` 版本号
  - `DESCRIPTION`  项目描述
  - `HOMEPAGE_URL`  项目主页URL
  - `LANGUAGES` 编程语言，默认为C/CXX

- 示例

  ```cmake
  project(MyLib
          VERSION 1.4.2
          DESCRIPTION "A demo library"
          HOMEPAGE_URL "https://github.com/example/mylib"
          LANGUAGES C CXX)
  ```

---

#### add_library()

##### 普通库

```cmake
# 使用指定的源文件向工程中添加一个目标库
add_library(<target> 
			[STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```

- 参数说明

  - `target` 库名称 (必须全局唯一)

   - `STATIC | SHARED | MODULE` 库类型，默认静态库
   - `EXCLUDE_FROM_ALL` 是否排除构建，如果增加这个属性，在默认编译时不会被编译
   - `source` 源文件列表

- 注意

  - 构建库的源文件可以直接指定，也可以后续使用 `target_sources()` 指定

  - 库类型：`STATIC` 静态库；`SHARED` 动态库； `MODULE` 插件模块

- 示例

  ```cmake
  add_library(libA STATIC src/libA.cpp)
  ```



##### 对象库

```cmake
# 编译 `source` 列表中的文件，仅生成目标文件（`.o`/`.obj`）,不将生成的目标文件打包/链接为库
add_library(<target> OBJECT [<source>...])
```

- 参数说明：
  - `target` 对象库名称 (必须全局唯一)
  - `source` 源文件列表

- 示例

  ```cmake
  add_library(libA OBJECT src/libA.cpp) # 仅编译文件，不将生成的目标文件打包/链接成库
  ```



##### 接口库

```cmake
# 生成一个接口库，这类库不编译任何文件，也不在磁盘上产生库文件
add_library(<target> INTERFACE [IMPORTED [GLOBAL]])
```

- 有一些属性被设置，并且能够被安装和导出。通常，使用以下命令在接口目标上填充属性：

  ```cmake
  set_property()
  target_link_library(INTERFACE)
  target_link_options(INTERFACE)
  target_include_directions(INTERFACE)
  target_compile_options(INTERFACE)
  target_compile_definitions(INTERFACE)
  target_sources(INTERFACE)
  ```



##### 导入库

```cmake
# 用来导入已经存在的库，CMake也不会添加任何编译规则给它
add_library(<target> <SHARED|STATIC|MODULE|UNKNOWN> IMPORTED [GLOBAL])
```

- 此类库的标志就是有IMPORT属性，导入的库的作用域为创建它的目录及更下级目录。但是如果有GLOBE属性，则作用域被拓展到全工程；
- 导入的库的类型必须是 `STATIC, SHARED, MODULE, UNKNOWN` 中的一种；
- 从工程外部引入一个库，使用 `IMPORTED_LOCATION` 属性确定库文件的在磁盘上的完整路径。



##### 别名库

```cmake
# 为给定库添加一个别名，后续可使用<name>来替代<target>
add_library(<name> ALIAS <target>)
```

---

 #### add_executable()

```cmake
# 使用指定的源文件创建出一个可执行文件
add_executable(<target> <source_files>...)
```

- 参数说明

  - `target` 可执行文件名称 (必须全局唯一)
  - `source` 源文件列表

- 示例

  ```cmake
  add_executable(MyExecutable main.cpp other_file.cpp)
  ```

---

#### target_include_directories()

```cmake
# 为目标（如可执行文件或库）指定包含目录（头文件搜索路径）
target_include_directories(<target>
                          [BEFORE | AFTER]
                          [SYSTEM]
                          [PUBLIC | PRIVATE | INTERFACE]
                          [items1...])
```

- 参数说明

  - `target` 需要添加包含目录的目标名称（例如通过 `add_executable` 或 `add_library` 创建的目标）。
  - `BEFORE| AFTER` 可选参数，控制新添加的包含目录是插入到现有列表的前面（`BEFORE`）还是后面（`AFTER`），默认为 `AFTER`
  - `SYSTEM` 可选参数，将指定的目录标记为“系统包含目录”
  - `PUBLIC | PRIVATE | INTERFACE` 必须三选一
    - `PUBLIC`：包含目录既用于编译目标本身，也会传递给链接该目标的其他目标。
    - `PRIVATE`：包含目录仅用于编译目标本身，不传递给其他目标。
    - `INTERFACE`：包含目录不用于编译目标本身，但会传递给链接该目标的其他目标（适用于头文件库）。
  - `<items>...` 要添加的包含目录路径（可以是绝对路径或相对路径）。支持生成器表达式（如 `$<BUILD_INTERFACE:...>`）。

- 示例

  ```cmake
  target_include_directories(MyExecutable PRIVATE ${PROJECT_SOURCE_DIR}/include)
  ```

---

#### target_link_libraries()

```cmake
# 为目标（如可执行文件或库）指定链接库
target_link_libraries(<target>
                      <PRIVATE|PUBLIC|INTERFACE> <item>...
                     [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

- 参数说明

  - `target`  需要链接库的目标名称
  - `作用域限定符` （必须）：
    - `PRIVATE`：库仅用于链接当前目标，不会传递给依赖该目标的其他目标。
    - `PUBLIC`：库既用于链接当前目标，也会传递给依赖该目标的其他目标。
    - `INTERFACE`：库不用于链接当前目标，但会传递给依赖该目标的其他目标（适用于头文件库）。
  - `<item>...`  可以是以下类型：
    - 库目标名称（通过 `add_library` 创建的目标，如 `my_lib`）。
    - 完整库文件路径（如 `/path/to/libfoo.a`）。
    - 库名称（如 `pthread`、`m` 等，编译器会自动添加 `-l` 前缀）。
    - 链接标志（如 `-framework Accelerate`）。
    - 生成器表达式（如 `$<LINK_LIBRARY:WHOLE_ARCHIVE,my_lib>`）。

- 示例

  ```cmake
  add_executable(my_app main.cpp)
  target_link_libraries(my_app PRIVATE pthread m)  # 链接pthread和math库
  ```

---

#### set()

```cmake
# 定义、修改或取消设置变量
set(<variable> <value>... [PARENT_SCOPE])
```

- 参数说明

  - `variable` 要被赋值/修改的变量
  - `value` 要赋给变量的值，多个值以分号 `;` 分隔的形式存储为列表
  - `PARENT_SCOPE`（可选）如果指定，变量将在父作用域中设置（例如在函数或宏中修改调用方的变量）

- 示例

  ```cmake
  set(MY_VAR "Hello World")        # 设置字符串
  set(ENABLE_FEATURE ON)           # 设置布尔值
  set(NUMBER 42)                   # 设置数字
  
  set(SOURCES main.cpp utils.cpp helper.cpp)  # 存储为 "main.cpp;utils.cpp;helper.cpp"
  set(INCLUDE_DIRS include src/lib)           # 多个目录路径
  ```

---

#### find_package()

```cmake
find_package(Boost REQUIRED)
# 指定版本
find_package(Boost 1.70 REQUIRED)
# 查找库并指定路径
find_package(OpenCV REQUIRED PATHS /path/to/opencv)
# 使用查找到的库
target_link_libraries(MyExecutable Boost::Boost)
```

---

#### install()

```cmake
# 指定在 make install（或等效命令）时如何安装目标文件（如可执行文件、库文件等）
install(TARGETS target1 [target2 ...]
		[EXPORT <export-name>]
        [RUNTIME DESTINATION dir]
        [LIBRARY DESTINATION dir]
        [ARCHIVE DESTINATION dir]
        [INCLUDES DESTINATION [dir ...]]
        [PRIVATE_HEADER DESTINATION dir]
        [PUBLIC_HEADER DESTINATION dir])
```

- 参数说明

  - **`<target1> [<target2> ...]`**
    指定要安装的一个或多个目标（通过 `add_executable` 或 `add_library` 创建）
  - **EXPORT：** 该参数用于将TARGETS进行导出，从而其他项目可以链接使用。
  - **`RUNTIME DESTINATION <dir>`**
    安装**可执行文件**（如 `.exe`、无扩展名的二进制文件）到指定目录。通常用于 `add_executable` 创建的目标。
  - **`LIBRARY DESTINATION <dir>`**
    安装**共享库**（如 `.so`、`.dll`、`.dylib`）到指定目录。通常用于 `SHARED` 库。
  - **`ARCHIVE DESTINATION <dir>`**
    安装**静态库**（如 `.a`、`.lib`）到指定目录。通常用于 `STATIC` 库。
  - **`INCLUDES DESTINATION [<dir> ...]`**
    安装目标相关的头文件目录（通常与 `PUBLIC_HEADER` 或 `PRIVATE_HEADER` 配合使用）。
  - **`PUBLIC_HEADER DESTINATION <dir>`**
    安装目标的公有头文件（通过 `PUBLIC_HEADER` 属性设置的文件）。
  - **`PRIVATE_HEADER DESTINATION <dir>`**
    安装目标的私有头文件（通过 `PRIVATE_HEADER` 属性设置的文件）。
  - **`RESOURCE DESTINATION <dir>`**
    安装目标相关的资源文件（如图像、配置文件等）。

- 示例

  ```cmake
  install(TARGETS MyExecutable RUNTIME DESTINATION bin) # 安装可执行文件到 bin 目录
  
  # 安装静态库和共享库
  install(TARGETS my_static_lib my_shared_lib
          ARCHIVE DESTINATION lib    # 静态库 (.a/.lib)
          LIBRARY DESTINATION lib    # 共享库 (.so/.dylib)
          RUNTIME DESTINATION bin    # Windows 的 .dll 文件（通常放在bin）
  )
  
  # 安装库及其头文件
  add_library(math STATIC math.cpp)
  target_include_directories(math PUBLIC include)
  install(TARGETS math
          ARCHIVE DESTINATION lib
          PUBLIC_HEADER DESTINATION include
          INCLUDES DESTINATION include
  )
  ```

---

#### 条件语句

```cmake
if(expression)
  # Commands
elseif(expression)
  # Commands
else()
  # Commands
endif()
```

---

#### 自定义命令

```cmake
add_custom_command(
   TARGET target
   PRE_BUILD | PRE_LINK | POST_BUILD
   COMMAND command1 [ARGS] [WORKING_DIRECTORY dir]
   [COMMAND command2 [ARGS]]
   [DEPENDS [depend1 [depend2 ...]]]
   [COMMENT comment]
   [VERBATIM]
)
```

- 参数说明

  - **`TARGET <target>`**

    指定此自定义命令要附加到哪个目标上。这个目标必须是由 `add_executable()` 或 `add_library()` 在**同一目录**中创建的。

  - **执行时机（三选一）：**

    - `PRE_BUILD`：在目标开始构建之前执行。
      - **注意**：在 Visual Studio 生成器中，它会在编译源文件之前运行，但在其他生成器（如 Makefile）中，其行为可能与 `PRE_LINK` 非常相似。
    - `PRE_LINK`：在编译完所有源文件之后，链接目标之前执行。这是最常用的时机，例如用于生成在链接阶段需要使用的源文件。
    - `POST_BUILD`：在目标构建完成之后执行。这是最常见的时机，例如用于打包、复制生成的可执行文件到其他目录、运行测试等。

  - **`COMMAND`**

    - 要执行的命令。可以是系统命令（如 `copy`, `md5sum`）、脚本解释器（如 `python`, `bash`）或任何其他可执行程序。
    - 可以指定多个 `COMMAND` 参数，它们会按顺序执行。

  - **`DEPENDS`**

    - 指定此自定义命令所依赖的文件或其他目标。如果这些依赖项比命令的输出文件新，命令就会重新执行。

  - **`COMMENT`**

    - 在构建时输出到终端的信息，用于说明正在执行什么操作，非常有用。

  - **`VERBATIM`**

    - **强烈建议始终使用**。它指示 CMake 将所有参数原封不动地传递给命令，防止在不同平台（如 Windows 和 Unix）上发生转义错误。

  - **`BYPRODUCTS`**

    - 指定命令会生成的文件。这对于像 Ninja 这样需要明确知道所有输入输出文件的构建系统至关重要，可以避免因找不到“隐式”输出文件而报错。

- 示例

  ```cmake
  # 在构建后复制可执行文件（POST_BUILD）
  add_executable(MyApp main.cpp)
  
  add_custom_command(TARGET MyApp
     POST_BUILD # 在 MyApp 构建成功后执行
     COMMAND ${CMAKE_COMMAND} -E copy # 使用 CMake 自带的跨平台 copy 命令
             $<TARGET_FILE:MyApp>    # 生成器表达式：获取 MyApp 的完整路径
             ${CMAKE_BINARY_DIR}/deploy/bin/
     COMMENT "Copying executable to deploy directory"
     VERBATIM
  )
  ```

  ```cmake
  # 在链接前生成一个源文件（PRE_LINK）
  add_executable(MyApp main.cpp)
  
  # 假设我们有一个脚本能生成 version.cpp
  add_custom_command(TARGET MyApp
     PRE_LINK    # 在链接之前执行，确保 version.cpp 已生成并可编译
     COMMAND python3 ${CMAKE_CURRENT_SOURCE_DIR}/generate_version.py
             --output ${CMAKE_CURRENT_BINARY_DIR}/version.cpp
     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
     COMMENT "Generating version source file"
     VERBATIM
  )
  
  # 需要将生成的源文件也添加到目标中
  target_sources(MyApp PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)
  ```

  ```cmake
  # 执行多个命令并添加依赖（POST_BUILD）
  add_executable(MyApp main.cpp)
  
  add_custom_command(TARGET MyApp
     POST_BUILD
     COMMAND echo "Starting post-build steps..."
     COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/package
     COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:MyApp> ${CMAKE_BINARY_DIR}/package/
     DEPENDS some_dependency.txt # 如果此文件更新，则重新运行此命令
     COMMENT "Packaging the application"
     VERBATIM
  )
  ```

  - 注意事项：

    1. **作用域**：这种形式的 `add_custom_command` 必须与 `TARGET` 在**同一个目录**的 `CMakeLists.txt` 中定义。

    2. **与 OUTPUT 形式的区别**：`add_custom_command` 还有另一种形式是 `OUTPUT`，它用于**生成文件**，并可以作为其他目标的依赖源。而 `TARGET` 形式则是将命令**绑定到构建过程的一个阶段**。

    3. **跨平台**：在 `COMMAND` 中尽量使用 `CMAKE_COMMAND -E` 提供的跨平台工具（如 `copy`, `remove`, `make_directory`），或者确保你的脚本和命令在所有目标平台上都可用。

       简单来说，**当你需要在某个特定目标的构建前、链接前或构建后自动执行一些操作时，就应该使用 `add_custom_command(TARGET ...)`**。

---

