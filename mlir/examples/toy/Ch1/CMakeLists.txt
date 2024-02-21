cmake_minimum_required(VERSION 3.28.3)

project(Toy)

set(CMAKE_CXX_STANDARD 17)


find_package(LLVM REQUIRED)
message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

set(LLVM_COMPONENTS core support mcjit orcjit native asmparser asmprinter)
list(APPEND LLVM_COMPONENTS aarch64info aarch64utils aarch64desc aarch64asmparser aarch64codegen aarch64disassembler)
llvm_map_components_to_libnames(llvm_libs ${LLVM_COMPONENTS})
add_library(p::llvm INTERFACE IMPORTED)
set_property(TARGET p::llvm PROPERTY INTERFACE_INCLUDE_DIRECTORIES
    ${LLVM_INCLUDE_DIRS})
set_property(TARGET p::llvm PROPERTY INTERFACE_COMPILE_OPTIONS
    $<$<COMPILE_LANGUAGE:CXX>:${LFORTRAN_CXX_NO_RTTI_FLAG}>)
set_property(TARGET p::llvm PROPERTY INTERFACE_LINK_LIBRARIES
    ${llvm_libs})





#-------------------------------------------------------------------------------

set(SRC
  toyc.cpp
  parser/AST.cpp
  )

add_executable(toyc-ch1 ${SRC})
target_include_directories(toyc-ch1 PRIVATE include/)
target_include_directories(toyc-ch1 PRIVATE $ENV{CONDA_PREFIX}/include)
target_link_libraries(toyc-ch1 PRIVATE p::llvm)
