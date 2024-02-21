cmake_minimum_required(VERSION 3.28.3)

project(Toy)

set(CMAKE_CXX_STANDARD 17)

#-------------------------------------------------------------------------------

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
#message("LLVM LIBS: ${llvm_libs}")

set(mlir_components
    Analysis
    AsmParser
    BytecodeReader
    IR
    ComplexDialect
    Parser
    SideEffectInterfaces
    Support
    Transforms)
foreach(LIB_NAME IN LISTS mlir_components)
    set(LIB_PATH "${LLVM_LIBRARY_DIRS}/libMLIR${LIB_NAME}.a")
    list(APPEND mlir_libs ${LIB_PATH})
endforeach()
add_library(p::mlir INTERFACE IMPORTED)
set_property(TARGET p::mlir PROPERTY INTERFACE_INCLUDE_DIRECTORIES
    ${LLVM_INCLUDE_DIRS})
set_property(TARGET p::mlir PROPERTY INTERFACE_COMPILE_OPTIONS
    $<$<COMPILE_LANGUAGE:CXX>:${LFORTRAN_CXX_NO_RTTI_FLAG}>)
set_property(TARGET p::mlir PROPERTY INTERFACE_LINK_LIBRARIES
    ${mlir_libs})
#message("MLIR LIBS: ${mlir_libs}")



#-------------------------------------------------------------------------------

execute_process(
    COMMAND mlir-tblgen
        -gen-dialect-decls
        -I$ENV{CONDA_PREFIX}/include/
        include/toy/Ops.td
        -o include/toy/Dialect.h.inc
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    RESULT_VARIABLE STATUS)
if (STATUS AND NOT STATUS EQUAL 0)
    message(FATAL_ERROR "mlir-tblgen failed with status: ${STATUS}")
else()
    message(STATUS "Done.")
endif()
execute_process(
    COMMAND mlir-tblgen
        -gen-dialect-defs
        -I$ENV{CONDA_PREFIX}/include/
        include/toy/Ops.td
        -o include/toy/Dialect.cpp.inc
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    RESULT_VARIABLE STATUS)
if (STATUS AND NOT STATUS EQUAL 0)
    message(FATAL_ERROR "mlir-tblgen failed with status: ${STATUS}")
else()
    message(STATUS "Done.")
endif()

execute_process(
    COMMAND mlir-tblgen
        --gen-op-decls
        -I$ENV{CONDA_PREFIX}/include/
        include/toy/Ops.td
        -o include/toy/Ops.h.inc
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    RESULT_VARIABLE STATUS)
if (STATUS AND NOT STATUS EQUAL 0)
    message(FATAL_ERROR "mlir-tblgen failed with status: ${STATUS}")
else()
    message(STATUS "Done.")
endif()
execute_process(
    COMMAND mlir-tblgen
        --gen-op-defs
        -I$ENV{CONDA_PREFIX}/include/
        include/toy/Ops.td
        -o include/toy/Ops.cpp.inc
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    RESULT_VARIABLE STATUS)
if (STATUS AND NOT STATUS EQUAL 0)
    message(FATAL_ERROR "mlir-tblgen failed with status: ${STATUS}")
else()
    message(STATUS "Done.")
endif()


set(SRC
  toyc.cpp
  parser/AST.cpp
  mlir/MLIRGen.cpp
  mlir/Dialect.cpp
  )

add_executable(toyc-ch2 ${SRC})
target_include_directories(toyc-ch2 PRIVATE include/)
target_include_directories(toyc-ch2 PRIVATE $ENV{CONDA_PREFIX}/include)
target_link_libraries(toyc-ch2 PRIVATE p::mlir p::llvm)
