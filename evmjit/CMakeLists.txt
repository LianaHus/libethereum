cmake_minimum_required(VERSION 3.0.0)

cmake_policy(SET CMP0042 OLD) 	# fix MACOSX_RPATH
cmake_policy(SET CMP0048 NEW) 	# allow VERSION argument in project()

project(EVMJIT VERSION 0.9.0.2 LANGUAGES CXX)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_AUTOMOC OFF)

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC")
else()
	set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Wextra -Wconversion -Wno-sign-conversion -Wno-unknown-pragmas ${CMAKE_CXX_FLAGS}")
endif()

if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND NOT SANITIZE)
	# Do not allow unresolved symbols in shared library (default on linux)
	# unless sanitizer is used (sanity checks produce unresolved function calls)
	set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined")
endif()

# LLVM
if (DEFINED MSVC)
	set(LLVM_DIR "${ETH_CMAKE_DIR}/extdep/install/windows/x64/share/llvm/cmake")
elseif (DEFINED APPLE)
	set(LLVM_DIR "/usr/local/opt/llvm37/lib/llvm-3.7/share/llvm/cmake")
endif()

find_package(LLVM REQUIRED CONFIG)
if (${LLVM_VERSION} VERSION_LESS 3.7)
	message(FATAL_ERROR "Incompatible LLVM version ${LLVM_VERSION}")
endif()
message(STATUS "Found LLVM ${LLVM_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")
llvm_map_components_to_libnames(LLVM_LIBS core support mcjit x86asmparser x86codegen ipo)

add_subdirectory(libevmjit)

if (DEFINED MSVC)
	set(EVMJIT_DLLS_LOCAL $<TARGET_FILE:evmjit>)
	set(EVMJIT_DLLS optimized ${EVMJIT_DLLS_LOCAL} debug ${EVMJIT_DLLS_LOCAL} PARENT_SCOPE)
endif()
