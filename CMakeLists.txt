cmake_minimum_required(VERSION 3.21)
project(subsdk)

# Error if not using switch toolchain file
if (NOT SWITCH)
    message(FATAL_ERROR "Not targeting switch, make sure to specify -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain.cmake")
endif()

# We are modern (as of 2022)
set(CMAKE_C_STANDARD 17)
set(CMAKE_CXX_STANDARD 20)
enable_language(ASM C CXX)

# Title Id
add_compile_definitions(EXL_PROGRAM_ID=0x0100801011c3e000)

# Project sources
include_directories(${PROJECT_SOURCE_DIR}/src/)
file(GLOB_RECURSE SOURCES_ASM ${PROJECT_SOURCE_DIR}/src/*.s)
file(GLOB_RECURSE SOURCES_C ${PROJECT_SOURCE_DIR}/src/*.c)
file(GLOB_RECURSE SOURCES_CXX ${PROJECT_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE SOURCES_SLPATCH ${PROJECT_SOURCE_DIR}/patches/*.slpatch)

# Include nx tools
include(${CMAKE_SOURCE_DIR}/cmake/switch/SwitchTools.cmake)

# subsdk9
set(CMAKE_EXECUTABLE_SUFFIX ".elf")
add_executable(subsdk9 ${SOURCES_ASM} ${SOURCES_C} ${SOURCES_CXX})
macro(msys_to_cmake_path MsysPath ResultingPath)
  string(REGEX REPLACE "^/([a-zA-Z])/" "\\1:/" ${ResultingPath} "${MsysPath}")
endmacro()

## Prepare subsdk ld
set(SUBSDK_LD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/subsdk.ld)
execute_process(COMMAND uname OUTPUT_VARIABLE uname)
if (uname MATCHES "^MINGW")
    msys_to_cmake_path(${SUBSDK_LD_PATH} SUBSDK_LD_PATH)
endif ()
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/subsdk.specs.template ${CMAKE_CURRENT_BINARY_DIR}/subsdk.specs)

# Build nso
add_nso_target(subsdk9)
set_target_properties(subsdk9 PROPERTIES CONFIG_JSON "title.json")
__generate_npdm(subsdk9)
set_target_properties(subsdk9 PROPERTIES LINK_FLAGS "-specs ${CMAKE_BINARY_DIR}/subsdk.specs -g ${ARCH}")