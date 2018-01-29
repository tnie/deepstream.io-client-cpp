# 预备工作

1. 下载 [cmake 最新版本](https://cmake.org/download/)，推荐使用 [免安装版本 cmake-3.10.2-win64-x64.zip][6]

	为了使用方便也可将其 bin 目录添加到 path 系统环境变量。

2. 克隆本仓库（feature_Compatible-with-linux 分支）
3. 打开 `deepstream.io-client-cpp/tools-windows` ，将 win_flex_bison-latest.zip 解压到 win_flex_bison-latest
4. 从 [WinXXOpenSSL][1] 下载 [Win32OpenSSL-1_1_0g.exe][2] 和 [Win64OpenSSL-1_1_0g.exe][3] 安装。

	在 `deepstream.io-client-cpp/tools-windows` 目录中留有备份，但网站能够下载到最新版本。

## boost 库

1. 从 [boost 官方网站](http://www.boost.org) 下载 [boost_1_65_1.7z][4] 解压，修改 `deepstream.io-client-cpp/CMakeLists.txt` 文件中 `BOOST_ROOT` 变量

	```CMake
	set(BOOST_ROOT "${CMAKE_SOURCE_DIR}/../boost_1_65_1/")
	```

2. 参考 [boost 指导手册][5]，编译 boost 库。

	- 注意区分 32 位、64 位。
	- boost 的 b2 构建无法同时生成 32 位、64 位版本。
	- 编译目标 `stage/lib` 目录在重新编译时会被覆盖，要想同时保留两种版本，需要重命名目录。

	```Batchfile
	:: 32 位
	.\b2.exe --with-test --build-type=complete
	:: 64 位
	.\b2.exe --with-test address-model=64 --build-type=complete
	```

3. 调整 `deepstream.io-client-cpp/CMakeLists.txt` 文件中 `BOOST_LIBRARYDIR` 变量

	```CMake
	set(BOOST_LIBRARYDIR "${BOOST_ROOT}/stage/lib32-test-complete")
	if("${CMAKE_GENERATOR}" MATCHES "Win64")
		set(BOOST_LIBRARYDIR "${BOOST_ROOT}/stage/lib64-test-complete")
	endif()
	```

# 开始

使用 cmake 构建 msvc 解决方案。使用 msvc-2015 编译项目。

## cmake 构建

windows 上可以使用 cmake-gui 操作，也可以使用 cmake 命令行操作。cmake-gui 操作简单，且命令行更具通用性，这里只介绍后者。

- 32 位版本：在 deepstream.io-client-cpp 目录新建（已存在直接打开，并删除其中所有内容）msvc32 目录中执行 `cmake -G "Visual Studio 14" ..`
- 64 位版本：在 msvc64 目录中执行 `cmake -G "Visual Studio 14 Win64" ..`

## msvc-2015 编译

打开解决方案，将 ds-example 改为启动项目，指定调试参数 "ws://ip:port/deepstream"，编译并执行。

[1]:https://slproweb.com/products/Win32OpenSSL.html
[2]:https://slproweb.com/download/Win32OpenSSL-1_1_0g.exe
[3]:https://slproweb.com/download/Win64OpenSSL-1_1_0g.exe
[4]:https://dl.bintray.com/boostorg/release/1.65.1/source/boost_1_65_1.7z
[5]:http://www.boost.org/doc/libs/1_66_0/more/getting_started/windows.html
[6]:https://cmake.org/files/v3.10/cmake-3.10.2-win64-x64.zip