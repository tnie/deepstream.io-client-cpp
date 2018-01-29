cmake_minimum_required(VERSION 2.8.12)
cmake_policy(SET CMP0054 NEW)

project(deepstream.io-client-cpp)

enable_language(C)
enable_language(CXX)
# OPTION(BUILD_TESTING "disable build testing auto" OFF)
# enable_testing()
# include(CTest)
include(ExternalProject)

if(WIN32)
	set(FLEX_EXECUTABLE "${CMAKE_SOURCE_DIR}/tools-windows/win_flex_bison-latest/win_flex.exe")
	set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
	# set(OPENSSL_ROOT_DIR "C:/OpenSSL-Win32")
	if("${CMAKE_GENERATOR}" MATCHES "Win64")
		# set(OPENSSL_ROOT_DIR "C:/OpenSSL-Win64")
	endif()
endif(WIN32)

if(WIN32 AND BUILD_TESTING)
	set(BOOST_ROOT "${CMAKE_SOURCE_DIR}/../boost_1_65_1/")
	set(BOOST_LIBRARYDIR "${BOOST_ROOT}/stage/lib32-test-complete")
	if("${CMAKE_GENERATOR}" MATCHES "Win64")
		set(BOOST_LIBRARYDIR "${BOOST_ROOT}/stage/lib64-test-complete")
	endif()
endif(WIN32 AND BUILD_TESTING)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/modules)

configure_file(version.hpp.in ${CMAKE_BINARY_DIR}/include/deepstream/core/version.hpp @ONLY)

# Put the libaries and binaries that get built into directories at the
# top of the build tree rather than in hard-to-find leaf directories.

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

option(CCACHE "Use ccache if available" ON)
find_program(CCACHE_PROGRAM ccache)
if(CCACHE AND CCACHE_PROGRAM)
  message(STATUS "using ccache")
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

# We absolutely depend on C++11 support from the toolchain; exit if
# the toolchain doesn't support it.

include(CheckCCompilerFlag)
include(CheckCXXCompilerFlag)

if(NOT WIN32)
	CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
	if(COMPILER_SUPPORTS_CXX11)
	  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
	else()
	  message(FATAL_ERROR "Compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
	endif()

	CHECK_C_COMPILER_FLAG("-std=c11" COMPILER_SUPPORTS_C11)
	if(COMPILER_SUPPORTS_C11)
	  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")
	else()
	  message(FATAL_ERROR "Compiler ${CMAKE_C_COMPILER} has no C11 support.")
	endif()

	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wextra -Wall -pedantic -g")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra -Wall -pedantic -g")
endif(NOT WIN32)

if(NOT DEFINED(CMAKE_BUILD_TYPE) OR "Debug" STREQUAL "${CMAKE_BUILD_TYPE}")
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GLIBCXX_ASSERTIONS")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GLIBCXX_DEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GLIBCXX_DEBUG_PEDANTIC")
  endif()
endif()

if(BUILD_COVERAGE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} --coverage")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --coverage")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --coverage")
endif()

OPTION(BUILD_POCO "enable build poco auto" ON)
if(BUILD_POCO)
  externalproject_add(poco
    PREFIX thirdparty
	# URL https://github.com/tnie/poco/archive/feature_deepstream.io-client-cpp.zip
	# 使用 zip 格式 linux 无法解压，直接将 .zip 后缀名改成 .tar.gz
	URL https://github.com/tnie/poco/archive/feature_deepstream.io-client-cpp.tar.gz ## 使用 zip 格式 linux 无法解压
    URL_HASH MD5=17A9E27BB95DAF19329F5D0244DDEF5A
    CMAKE_GENERATOR ${CMAKE_GENERATOR}
    CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE}
    CMAKE_ARGS
    -DENABLE_MONGODB=OFF
    -DENABLE_PDF=OFF
    -DENABLE_DATA=OFF
    -DENABLE_DATA_SQLITE=OFF
    -DENABLE_DATA_MYSQL=OFF
    -DENABLE_DATA_ODBC=OFF
    -DENABLE_SEVENZIP=OFF
    -DENABLE_ZIP=OFF
    -DENABLE_APACHECONNECTOR=OFF
    -DENABLE_CPPPARSER=OFF
    -DENABLE_POCODOC=OFF
    -DENABLE_PAGECOMPILER=OFF
    -DENABLE_PAGECOMPILER_FILE2PAGE=OFF
    -DENABLE_JSON=OFF
    -DENABLE_XML=OFF
	# 会在 thirdparty 下新增 bin/lib/include 三个目录
	# msvc 不同编译模式下不会生成对应的子目录。安装本也不该再区分 debug/release 子目录。
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/thirdparty
	-DCMAKE_RUNTIME_OUTPUT_DIRECTORY=${CMAKE_BINARY_DIR}/bin
    LOG_CONFIGURE ON
    UPDATE_COMMAND "")

  # 此定义多余，在 windows 上造成 bug
  # set(Poco_LIBRARIES PocoNetSSL PocoCrypto PocoNet PocoUtil PocoFoundation)
else()
  # windows 下找不到包怎么不报错，不中断呢？在 linux 下找不到会中断。
  # 总而言之，结合官方库的脚本来看，原创作者就没打算走这个分支 find_package()，至少是还没调试好。 
  find_package(Poco 1.7.8 REQUIRED NetSSL Crypto Net Util Foundation)
  message(STATUS "#### POCO_FOUND=${POCO_FOUND}")
endif()

find_package(SWIG 3)
find_package(FLEX 2.5 REQUIRED)

link_directories(${CMAKE_BINARY_DIR}/thirdparty/lib)
# 全局作用域
include_directories(${CMAKE_BINARY_DIR}/thirdparty/include)

include_directories(${CMAKE_SOURCE_DIR}/include ${CMAKE_BINARY_DIR}/include)

option(BUILD_SWIG "Build the SWIG python wrapper" OFF)
if(BUILD_SWIG AND SWIG_FOUND)
  # We need to fix the generated Swig wrapper as the current version
  # of Swig does not handle std::unique_ptr. Our fix directly calls
  # std::move() for the few cases where this smart pointer type is
  # exposed in the deepstream public API.

  include(${SWIG_USE_FILE})
  find_package(PythonLibs REQUIRED)
  include_directories(${PYTHON_INCLUDE_PATH})
  set(CMAKE_SWIG_FLAGS "")
  set(CMAKE_SWIG_OUTDIR python)
  set_source_files_properties(swig/deepstream.i PROPERTIES CPLUSPLUS ON)
  set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/python/deepstreamPYTHON_wrap.cxx PROPERTIES COMPILE_FLAGS -Wno-self-assign)
  swig_add_module(deepstream python swig/deepstream.i)
  swig_link_libraries(deepstream libdeepstream_core ${POCO_LIBS} ${PYTHON_LIBRARIES})

  add_test(
    NAME    python-swig-test
    COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_SOURCE_DIR}/test/swig/python-test.py
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endif()

add_subdirectory(doc)
add_subdirectory(src/core)
add_subdirectory(src/lib)
add_subdirectory(examples)

if(BUILD_TESTING)
  find_package(Boost 1.46 REQUIRED COMPONENTS unit_test_framework)
  if(Boost_FOUND)
      add_subdirectory(test/core)
      add_subdirectory(test/lib)
  endif()
endif()

install(
    DIRECTORY ${CMAKE_SOURCE_DIR}/include/
    DESTINATION include
    FILES_MATCHING PATTERN "*.hpp"
)

install(
    FILES ${CMAKE_BINARY_DIR}/include/deepstream/core/version.hpp
    DESTINATION include/deepstream/core
)

message(STATUS "CMAKE_SYSTEM=${CMAKE_SYSTEM}")
message(STATUS "CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}")
message(STATUS "CMAKE_SYSTEM_PROCESSOR=${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "CMAKE_GENERATOR=${CMAKE_GENERATOR}")
message(STATUS "CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
message(STATUS "CMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
message(STATUS "CMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")

message(STATUS "BUILD_TESTING=${BUILD_TESTING}")
message(STATUS "Boost_FOUND=${Boost_FOUND}")
message(STATUS "BUILD_COVERAGE=${BUILD_COVERAGE}")
message(STATUS "BUILD_POCO=${BUILD_POCO}")
message(STATUS "Poco_LIBRARIES=${Poco_LIBRARIES}")

message(STATUS "OPENSSL_ROOT_DIR is:${OPENSSL_ROOT_DIR}")
message(STATUS "BOOST_LIBRARYDIR is:${BOOST_LIBRARYDIR}")

# debug
message(STATUS "#### POCO_INCLUDE_DIR=${POCO_INCLUDE_DIR}")
message(STATUS " POCO_INCLUDE_DIRS=${POCO_INCLUDE_DIRS}")
message(STATUS "#### POCO_LIBRARY=${POCO_LIBRARY}")
message(STATUS " POCO_LIBRARYS=${POCO_LIBRARYS}")
message(STATUS "#### Poco_LIBRARIES=${Poco_LIBRARIES}")
