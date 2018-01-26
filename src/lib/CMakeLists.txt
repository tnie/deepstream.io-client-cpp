set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
find_package(OpenSSL REQUIRED)

add_library(
  libdeepstream_poco STATIC
  poco-ws.cpp)

set_target_properties(libdeepstream_poco PROPERTIES OUTPUT_NAME deepstream-poco)
# TODO：PUBLIC 属性决定了 install 时会安装的头文件、库文件？
message(STATUS "#### NON_EXIST=${NON_EXIST}")
message(STATUS "#### POCO_INCLUDE_DIR=${POCO_INCLUDE_DIR}")
# 此目录 POCO_INCLUDE_DIR 多余。
target_include_directories(libdeepstream_poco PUBLIC ${CMAKE_CURRENT_BINARY_DIR} ${OPENSSL_INCLUDE_DIR} )#${POCO_INCLUDE_DIR})
message(STATUS "#### Poco_LIBRARIES=${Poco_LIBRARIES}")
# TODO：变量 Poco_LIBRARIES 多余？怎么指定连接库的呢？
target_link_libraries(libdeepstream_poco PUBLIC libdeepstream_core ${OPENSSL_LIBRARIES} Threads::Threads )#${Poco_LIBRARIES})
install(TARGETS libdeepstream_poco DESTINATION "lib")

if(BUILD_POCO)
  add_dependencies(libdeepstream_poco poco)
endif()
add_dependencies(libdeepstream_poco libdeepstream_core)

if(BUILD_TESTING AND Boost_FOUND)
  add_subdirectory(test)
endif()
