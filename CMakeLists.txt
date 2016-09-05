#CC=clang CXX=clang++ cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/zks/local/ ../libzks
cmake_minimum_required (VERSION 2.6) 
project(fastcgi)

if(NOT CMAKE_BUILD_TYPE)
    set( CMAKE_BUILD_TYPE Release )
endif()

string(CONCAT CMAKE_CXX_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG} " -DDEBUG -D_DEBUG")
string(CONCAT CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE} " -D_NDEBUG")

add_compile_options ($<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:GNU>>:-std=c++1y>)
add_compile_options ($<$<CXX_COMPILER_ID:Clang>:-stdlib=libc++>)

string( TOLOWER ${CMAKE_BUILD_TYPE} CONFIG )
set( CMAKE_DEBUG_POSTFIX "_d" )

if(CMAKE_CXX_COMPILER_ID STREQUAL Clang)
	string(CONCAT CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} " -lc++")
endif()

if(MSVC)
	string(CONCAT CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} " -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS")
	foreach(flag_var
			CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
			CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
	   if(${flag_var} MATCHES "/MD")
		  string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
	   endif(${flag_var} MATCHES "/MD")
	endforeach(flag_var)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	string(CONCAT CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} " -framework CoreFoundation -framework IOKit")
endif()

MESSAGE(STATUS "CONFIG : ${CONFIG}")
MESSAGE(STATUS "CMAKE_BUILD_TYPE : ${CMAKE_BUILD_TYPE}")
MESSAGE(STATUS "CMAKE_CXX_COMPILER_ID : ${CMAKE_CXX_COMPILER_ID}")
MESSAGE(STATUS "CMAKE_CXX_FLAGS : ${CMAKE_CXX_FLAGS}")
MESSAGE(STATUS "CMAKE_CXX_FLAGS_DEBUG : ${CMAKE_CXX_FLAGS_DEBUG}")
MESSAGE(STATUS "CMAKE_CXX_FLAGS_RELEASE : ${CMAKE_CXX_FLAGS_RELEASE}")
MESSAGE(STATUS "CMAKE_STANDARD_LIBRARIES : ${CMAKE_STANDARD_LIBRARIES}")
MESSAGE(STATUS "CMAKE_EXE_LINKER_FLAGS : ${CMAKE_EXE_LINKER_FLAGS}")
MESSAGE(STATUS "CMAKE_DEBUG_POSTFIX : ${CMAKE_DEBUG_POSTFIX}")

set( CMAKE_DEBUG_TARGET_PROPERTIES on )
set( CMAKE_VERBOSE_MAKEFILE on )

find_package ( Threads REQUIRED )

file(GLOB FCGIXX_SRC "./libfcgi/fcgi*.cpp")
file(GLOB FCGI_SRC "./libfcgi/fcgi*.c")
if(WIN32)
	list(APPEND FCGI_SRC "./libfcgi/os_win32.c")
else(WIN32)
	list(APPEND FCGI_SRC "./libfcgi/os_unix.c")
endif()
file(GLOB FCGI_HEADER "./include/*.h")

add_library (fcgi STATIC ${FCGI_SRC} ${FCGI_HEADER})
add_library (fcgi++ STATIC ${FCGIXX_SRC} ${FCGI_HEADER})

target_include_directories(fcgi PUBLIC "./include/")
target_include_directories(fcgi++ PUBLIC "./include/")

install (TARGETS fcgi DESTINATION lib)
install (TARGETS fcgi++ DESTINATION lib)
install (FILES ${FCGI_HEADER} DESTINATION include)