if(NOT WIN32)
	add_custom_command(
		OUTPUT
			lexer.c
			lexer.h
		DEPENDS ${CMAKE_SOURCE_DIR}/src/core/lexer.l
		COMMAND ${FLEX_EXECUTABLE}
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
			COMPILE_FLAGS "-Wno-unused-function -Wno-unused-parameter -Wno-type-limits -Wno-sign-compare"
	)
else()
	add_custom_command(
		OUTPUT
			lexer.c
			lexer.h
		DEPENDS ${CMAKE_SOURCE_DIR}/src/core/lexer.l
		COMMAND ${FLEX_EXECUTABLE}
			--nounistd
			--wincompat
			--outfile=lexer.c
			--header-file=lexer.h
			-- ${CMAKE_SOURCE_DIR}/src/core/lexer.l
		COMMENT "[FLEX][src] Building lexer with Flex ${FLEX_VERSION}"
		VERBATIM
	)
endif(NOT WIN32)

add_library(
    libdeepstream_core STATIC
    client.cpp
    event.cpp
    exception.cpp
    connection.cpp
    message.cpp
    message_builder.cpp
    message_proxy.cpp
    parser.cpp
    presence.cpp
    random.cpp
    "${CMAKE_CURRENT_BINARY_DIR}/lexer.c")

set_target_properties(libdeepstream_core PROPERTIES OUTPUT_NAME deepstream-core)
target_include_directories(libdeepstream_core PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
install(TARGETS libdeepstream_core DESTINATION "lib")
