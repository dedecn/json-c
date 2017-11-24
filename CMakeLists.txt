#Licensed under the MIT license. See LICENSE file in the project root for full license information.

cmake_minimum_required(VERSION 2.8.7)
project(json-c)

if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4100 /wd4996 /wd4244 /wd4706 /wd4702 /wd4127 /wd4701")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4100 /wd4996 /wd4244 /wd4706 /wd4702 /wd4127 /wd4701")
  add_definitions(-Dinline=__inline)
elseif(UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror")
endif()

add_definitions(-D_GNU_SOURCE)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

# perform platform checks
# For more information on performing checks see: https://cmake.org/Wiki/CMake:How_To_Write_Platform_Checks
INCLUDE (CheckIncludeFiles)
INCLUDE (CheckFunctionExists)
INCLUDE (CheckLibraryExists)
INCLUDE (CheckSymbolExists)
INCLUDE (CheckTypeSize)

CHECK_INCLUDE_FILES (dlfcn.h HAVE_DLFCN_H)
CHECK_INCLUDE_FILES (endian.h HAVE_ENDIAN_H)
CHECK_INCLUDE_FILES (fcntl.h HAVE_FCNTL_H)
CHECK_INCLUDE_FILES (inttypes.h HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES (inttypes.h JSON_C_HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES (limits.h HAVE_LIMITS_H)
CHECK_INCLUDE_FILES (locale.h HAVE_LOCALE_H)
CHECK_INCLUDE_FILES (memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILES (stdarg.h HAVE_STDARG_H)
CHECK_INCLUDE_FILES (stdint.h HAVE_STDINT_H)
CHECK_INCLUDE_FILES (stdlib.h HAVE_STDLIB_H)
CHECK_INCLUDE_FILES (strings.h HAVE_STRINGS_H)
CHECK_INCLUDE_FILES (string.h HAVE_STRING_H)
CHECK_INCLUDE_FILES (syslog.h HAVE_SYSLOG_H)
CHECK_INCLUDE_FILES (sys/cdefs.h HAVE_SYS_CDEFS_H)
CHECK_INCLUDE_FILES (sys/param.h HAVE_SYS_PARAM_H)
CHECK_INCLUDE_FILES (sys/stat.h HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILES (sys/types.h HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES (unistd.h HAVE_UNISTD_H)


CHECK_SYMBOL_EXISTS(malloc "stdlib.h;memory.h" HAVE_MALLOC)
CHECK_SYMBOL_EXISTS(realloc "stdlib.h;memory.h" HAVE_REALLOC)
CHECK_FUNCTION_EXISTS(open HAVE_OPEN)
CHECK_FUNCTION_EXISTS(_doprnt HAVE_DOPRNT)

CHECK_SYMBOL_EXISTS(isnan "math.h" HAVE_DECL_ISNAN)
CHECK_SYMBOL_EXISTS(NAN "math.h" HAVE_DECL_NAN)
CHECK_SYMBOL_EXISTS(_isnan "float.h" HAVE_DECL__ISNAN)
CHECK_SYMBOL_EXISTS(_finite "float.h" HAVE_DECL__FINITE)
CHECK_SYMBOL_EXISTS(isinf "math.h" HAVE_DECL_ISINF)
CHECK_SYMBOL_EXISTS(INFINITY "math.h" HAVE_DECL_INFINITY)

CHECK_TYPE_SIZE("int" SIZEOF_INT)

CHECK_SYMBOL_EXISTS(strdup "string.h" HAVE_STRDUP)
CHECK_SYMBOL_EXISTS(strncasecmp "string.h" HAVE_STRNCASECMP)
CHECK_SYMBOL_EXISTS(strcasecmp "string.h" HAVE_STRCASECMP)
CHECK_LIBRARY_EXISTS(c strerror "string.h" HAVE_STRERROR)

CHECK_LIBRARY_EXISTS(c realloc "stdlib.h" HAVE_REALLOC)
CHECK_LIBRARY_EXISTS(c malloc "stdlib.h" HAVE_MALLOC)
CHECK_LIBRARY_EXISTS(c setlocale "locale.h" HAVE_SETLOCALE)
CHECK_SYMBOL_EXISTS(snprintf "stdio.h" HAVE_SNPRINTF)
CHECK_SYMBOL_EXISTS(vprintf "stdio.h;stdarg.h" HAVE_VPRINTF)
CHECK_SYMBOL_EXISTS(vsnprintf "stdio.h" HAVE_VSNPRINTF)
CHECK_LIBRARY_EXISTS(c vasprintf "stdlib.h;stdio.h" HAVE_VASPRINTF)
CHECK_LIBRARY_EXISTS(c vsyslog "stdarg.h" HAVE_VSYSLOG)


CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/config.h)
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/cmake/json_config.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/json_config.h)

# This function takes at least 2 arguments and prepends (path) to each entry from 2-N
# It can be used to convert a list of relative file paths into absolute ones
FUNCTION(PREPEND var prefix)
   SET(listVar "")
   FOREACH(f ${ARGN})
      LIST(APPEND listVar "${prefix}/${f}")
   ENDFOREACH(f)
   SET(${var} "${listVar}" PARENT_SCOPE)
ENDFUNCTION(PREPEND)


set(JSON_C_HEADERS
    json.h
    json_c_version.h
    arraylist.h
    debug.h
    json_inttypes.h
    json_object.h
    json_object_private.h
    json_object_iterator.h
    json_pointer.h
    json_tokener.h
    json_util.h
    linkhash.h
    math_compat.h
    strdup_compat.h
    vasprintf_compat.h
    printbuf.h
    random_seed.h
)
set(JSON_C_GENERATED_HEADERS
    ${CMAKE_CURRENT_BINARY_DIR}/include/json_config.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/config.h
)

set(JSON_C_SOURCES
    ./arraylist.c
    ./debug.c
    ./json_object.c
    ./json_pointer.c
    ./json_tokener.c
    ./json_util.c
    ./linkhash.c
    ./printbuf.c
    ./random_seed.c
)

# we will need absolute paths to each header for some commands
PREPEND(JSON_C_HEADERS_ABSPATH ${CMAKE_CURRENT_SOURCE_DIR} ${JSON_C_HEADERS})


add_library(json-c-objlib OBJECT
  ${JSON_C_SOURCES}
  ${JSON_C_HEADERS}
  ${JSON_C_GENERATED_HEADERS}
)

# shared libraries need PIC
set_property(TARGET json-c-objlib PROPERTY POSITION_INDEPENDENT_CODE 1)
set_property(TARGET json-c-objlib PROPERTY C_STANDARD 99)

add_library(json-c-static STATIC $<TARGET_OBJECTS:json-c-objlib>)
add_library(json-c-shared SHARED $<TARGET_OBJECTS:json-c-objlib>)

SET_TARGET_PROPERTIES(json-c-static PROPERTIES OUTPUT_NAME json-c CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(json-c-shared PROPERTIES OUTPUT_NAME json-c CLEAN_DIRECT_OUTPUT 1)

# attach version number to SO shared file
set(MYLIB_VERSION_MAJOR 0)
set(MYLIB_VERSION_MINOR 12)
set(MYLIB_VERSION_PATCH 99)
set(MYLIB_VERSION_STRING ${MYLIB_VERSION_MAJOR}.${MYLIB_VERSION_MINOR}.${MYLIB_VERSION_PATCH})
set_target_properties(json-c-shared PROPERTIES VERSION ${MYLIB_VERSION_STRING}
                                           SOVERSION ${MYLIB_VERSION_MAJOR})


# copy required json assets to output directory
add_custom_command(TARGET json-c-shared POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${JSON_C_HEADERS_ABSPATH} ${CMAKE_CURRENT_BINARY_DIR}/include
        )

# installation locations
install(TARGETS json-c-static json-c-shared
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

#install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/json-c FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/ DESTINATION ${CMAKE_INSTALL_PREFIX}/include/json-c FILES_MATCHING PATTERN "*.h")
install(FILES ${JSON_C_HEADERS} ${JSON_C_GENERATED_HEADERS} DESTINATION include/json-c )

