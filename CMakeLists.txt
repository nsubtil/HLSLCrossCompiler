project(HLSLCrossCompiler)
cmake_minimum_required(VERSION 3.6)

#if (${CMAKE_BUILD_TYPE} STREQUAL Debug)
#    list(APPEND hlslcc_definitions -DDEBUG -D_DEBUG -D__DEBUG__)
#endif()

if (BIG_ENDIAN_SYSTEM)
    list(APPEND hlslcc_definitions -D__BIG_ENDIAN__)
else()
    list(APPEND hlslcc_definitions -D__LITTLE_ENDIAN__)
endif()

if (MSVC)
    option(USE_MSVC_FAST_FLOATINGPOINT "Use MSVC /fp:fast option" ON)
    if (USE_MSVC_FAST_FLOATINGPOINT)
        list(APPEND hlslcc_definitions /fp:fast)
    endif()
endif()

if (WIN32)
    list(APPEND hlslcc_definitions -D_CRT_SECURE_NO_WARNINGS /WX /W3 /arch:SSE2 /wd4091 -DEXCLUDE_PSTDINT)
endif()

list(APPEND hlslcc_include_directories_public ${CMAKE_CURRENT_SOURCE_DIR}/include)
list(APPEND hlslcc_include_directories_private src)

set(libcbstring_sources
#    src/cbstring/bsafe.c
#    src/cbstring/bsafe.h
    src/cbstring/bstraux.c
    src/cbstring/bstraux.h
    src/cbstring/bstrlib.c
    src/cbstring/bstrlib.h)

set(libcjson_sources
    offline/cjson/cJSON.c
    offline/cjson/cJSON.h)

set(libhlslcc_sources
    src/internal_includes/debug.h
    src/internal_includes/decode.h
    src/internal_includes/languages.h
    include/pstdint.h
    src/internal_includes/reflect.h
    src/internal_includes/structs.h
    include/hlslcc.h
    src/internal_includes/toGLSLDeclaration.h
    src/internal_includes/toGLSLInstruction.h
    src/internal_includes/toGLSLOperand.h
    src/internal_includes/tokens.h
    src/internal_includes/tokensDX9.h
    src/internal_includes/shaderLimits.h
    src/internal_includes/hlslcc_malloc.h
    src/internal_includes/hlslcc_malloc.c
    src/decode.c
    src/decodeDX9.c
    src/reflect.c
    src/toGLSL.c
    src/toGLSLInstruction.c
    src/toGLSLOperand.c
    src/toGLSLDeclaration.c)

add_library(libhlslcc STATIC EXCLUDE_FROM_ALL ${libhlslcc_sources} ${libcbstring_sources})
target_compile_definitions(libhlslcc PUBLIC ${hlslcc_definitions})
target_include_directories(libhlslcc PUBLIC ${hlslcc_include_directories_public})
target_include_directories(libhlslcc PRIVATE ${hlslcc_include_directories_private} src/cbstring)

set(hlslcc_sources
    offline/toGLSLStandalone.cpp
    offline/timer.h
    offline/timer.cpp
    offline/hash.h
    offline/serializeReflection.cpp
    offline/serializeReflection.h)

add_executable(hlslcc EXCLUDE_FROM_ALL ${hlslcc_sources} ${libcjson_sources})
target_include_directories(hlslcc PRIVATE offline/cjson)
target_link_libraries(hlslcc libhlslcc)
