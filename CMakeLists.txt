cmake_minimum_required(VERSION 3.21)
project(subsdk)

## Error if not using switch toolchain file
if (NOT SWITCH)
  message(FATAL_ERROR "Not targeting switch, make sure to specify -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain.cmake")
endif()

set(CMAKE_C_STANDARD 17)
set(CMAKE_CXX_STANDARD 20)
enable_language(ASM C CXX)

## Title Id
set(CONFIG_TITLE_ID 0x0100801011c3e000)
add_compile_definitions(EXL_PROGRAM_ID=${CONFIG_TITLE_ID} EXL_LOAD_KIND=Module EXL_LOAD_KIND_ENUM=2)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/module/subsdk9.json.template ${CMAKE_CURRENT_BINARY_DIR}/subsdk9.json)

## Project sources
include_directories(${PROJECT_SOURCE_DIR}/src/)
file(GLOB_RECURSE SOURCES_ASM ${PROJECT_SOURCE_DIR}/src/*.s)
file(GLOB_RECURSE SOURCES_C ${PROJECT_SOURCE_DIR}/src/*.c)
file(GLOB_RECURSE SOURCES_CXX ${PROJECT_SOURCE_DIR}/src/*.cpp)

## Include nx tools
include(${CMAKE_SOURCE_DIR}/cmake/SwitchTools.cmake)

## subsdk9
set(CMAKE_EXECUTABLE_SUFFIX ".elf")
add_executable(subsdk9 ${SOURCES_ASM} ${SOURCES_C} ${SOURCES_CXX})

## Prepare subsdk ld
set(SUBSDK_LD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/module/subsdk9.ld)
execute_process(COMMAND uname OUTPUT_VARIABLE uname)
if (uname MATCHES "^MINGW")
  string(REGEX REPLACE "^/([a-zA-Z])/" "\\1:/" SUBSDK_LD_PATH "${SUBSDK_LD_PATH}")
endif ()
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/module/subsdk9.specs.template ${CMAKE_CURRENT_BINARY_DIR}/subsdk9.specs)

## Build nso & generate npdm
add_nso_target_subsdk(subsdk9)
set_target_properties(subsdk9 PROPERTIES CONFIG_JSON "${CMAKE_CURRENT_BINARY_DIR}/subsdk9.json")
__generate_npdm(subsdk9)
set_target_properties(subsdk9 PROPERTIES LINK_FLAGS "-specs ${CMAKE_BINARY_DIR}/subsdk9.specs -g ${ARCH} -nostartfiles -nostdlib")