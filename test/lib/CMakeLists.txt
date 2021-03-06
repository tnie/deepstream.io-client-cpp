# This functions searches all unit tests implemented with boost.test and turns
# each of them into a single CTest test case. See `doc/ctest-vs-boost.md` for
# the motivation behind this.
# Source:
# https://eb2.co/blog/2015/06/driving-boost-dot-test-with-cmake/
function(add_boost_test FILENAME DEPENDENCY_LIB)
    get_filename_component(TEST_EXECUTABLE ${FILENAME} NAME_WE)
    add_executable(${TEST_EXECUTABLE} ${FILENAME})
    target_compile_definitions(${TEST_EXECUTABLE} PUBLIC -DBOOST_TEST_DYN_LINK)
    target_include_directories(${TEST_EXECUTABLE} PUBLIC ${Boost_INCLUDE_DIRS})
    target_link_libraries(${TEST_EXECUTABLE} ${DEPENDENCY_LIB})
    target_link_libraries(${TEST_EXECUTABLE} ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY})

    file(READ "${FILENAME}" SOURCE)
    string(
        REGEX MATCHALL
        "BOOST_AUTO_TEST_CASE\\( *([A-Za-z0-9_]+) *\\)"
        TESTS
        ${SOURCE})

    foreach(TEST ${TESTS})
        string(
            REGEX REPLACE
            ".*\\( *([A-Za-z0-9_]+) *\\).*" "\\1"
            TEST_NAME
            ${TEST})

        add_test(
            NAME "${TEST_EXECUTABLE}.${TEST_NAME}"
            COMMAND ${TEST_EXECUTABLE} --run_test=${TEST_NAME})
    endforeach()
endfunction()

add_custom_command(
    OUTPUT
        lexer.c
        lexer.h
    DEPENDS ${CMAKE_SOURCE_DIR}/src/core/lexer.l
    COMMAND ${FLEX_EXECUTABLE}
if(WIN32)
		--nounistd
		--wincompat
endif(WIN32)
        --outfile=lexer.c
        --header-file=lexer.h
        -- ${CMAKE_SOURCE_DIR}/src/core/lexer.l
    COMMENT "[FLEX][src] Building lexer with Flex ${FLEX_VERSION}"
    VERBATIM
)

set_source_files_properties(
    "${CMAKE_CURRENT_BINARY_DIR}/lexer.c"
    PROPERTIES
        COMPILE_DEFINITIONS "_POSIX_SOURCE"
if(NOT WIN32)
        COMPILE_FLAGS "-Wno-unused-function -Wno-unused-parameter -Wno-type-limits -Wno-sign-compare"
endif(NOT WIN32)
)

include_directories(${CMAKE_SOURCE_DIR})

add_library(libdeepstream_poco_test SHARED ../dummy.cpp)
set_target_properties(libdeepstream_poco_test PROPERTIES OUTPUT_NAME deepstream-poco-test)
target_link_libraries(libdeepstream_poco_test PUBLIC libdeepstream_poco)
install(TARGETS libdeepstream_poco_test DESTINATION "lib")

add_boost_test(test-serial.cpp libdeepstream_poco_test)
