cmake_minimum_required(VERSION 3.15)

project(swig VERSION 4.0.1 LANGUAGES C CXX)

find_program(BISON_COMMAND bison NAMES win_bison)
find_package(PCRE REQUIRED NO_MODULE)

add_custom_target(gen-parse ALL
  COMMAND
    ${BISON_COMMAND} -y -d -b parser ${CMAKE_CURRENT_SOURCE_DIR}/Source/CParse/parser.y
  WORKING_DIRECTORY
    ${PROJECT_BINARY_DIR}
  BYPRODUCTS
    ${PROJECT_BINARY_DIR}/parser.tab.h
    ${PROJECT_BINARY_DIR}/parser.tab.c
)

file(STRINGS Source/Include/swigwarn.h SWIGWARNINGS REGEX [[^#define WARN|/\*.*\*/|^[ \t]*$]])
list(APPEND SWIGOUT "/* SWIG warning codes - generated from swigwarn.h - do not edit */")
foreach(line ${SWIGWARNINGS})
  if(line MATCHES [[^#define (WARN.*[0-9][0-9]*)(.*)$]] )
    list(APPEND SWIGOUT "%define SWIG${CMAKE_MATCH_1} %enddef${CMAKE_MATCH_2}\n")
  else()
    list(APPEND SWIGOUT "${line}\n")
  endif()
endforeach()
file(WRITE Lib/swigwarn.swg ${SWIGOUT})

add_executable(swig
  Source/CParse/cscanner.c
  Source/CParse/parser.y
  Source/CParse/templ.c
  Source/CParse/util.c
  Source/DOH/base.c
  Source/DOH/file.c
  Source/DOH/fio.c
  Source/DOH/hash.c
  Source/DOH/list.c
  Source/DOH/memory.c
  Source/DOH/string.c
  Source/DOH/void.c
  Source/Doxygen/doxyentity.cxx
  Source/Doxygen/doxyentity.h
  Source/Doxygen/doxyparser.cxx
  Source/Doxygen/doxyparser.h
  Source/Doxygen/doxytranslator.cxx
  Source/Doxygen/doxytranslator.h
  Source/Doxygen/javadoc.cxx
  Source/Doxygen/javadoc.h
  Source/Doxygen/pydoc.cxx
  Source/Doxygen/pydoc.h
  Source/Modules/allocate.cxx
  Source/Modules/browser.cxx
  Source/Modules/contract.cxx
  Source/Modules/csharp.cxx
  Source/Modules/d.cxx
  Source/Modules/directors.cxx
  Source/Modules/emit.cxx
  Source/Modules/go.cxx
  Source/Modules/guile.cxx
  Source/Modules/interface.cxx
  Source/Modules/java.cxx
  Source/Modules/javascript.cxx
  Source/Modules/lang.cxx
  Source/Modules/lua.cxx
  Source/Modules/main.cxx
  Source/Modules/mzscheme.cxx
  Source/Modules/nested.cxx
  Source/Modules/ocaml.cxx
  Source/Modules/octave.cxx
  Source/Modules/overload.cxx
  Source/Modules/perl5.cxx
  Source/Modules/php.cxx
  Source/Modules/python.cxx
  Source/Modules/r.cxx
  Source/Modules/ruby.cxx
  Source/Modules/scilab.cxx
  Source/Modules/swigmain.cxx
  Source/Modules/tcl8.cxx
  Source/Modules/typepass.cxx
  Source/Modules/utils.cxx
  Source/Modules/xml.cxx
  Source/Preprocessor/cpp.c
  Source/Preprocessor/expr.c
  Source/Swig/cwrap.c
  Source/Swig/deprecate.c
  Source/Swig/error.c
  Source/Swig/extend.c
  Source/Swig/fragment.c
  Source/Swig/getopt.c
  Source/Swig/include.c
  Source/Swig/misc.c
  Source/Swig/naming.c
  Source/Swig/parms.c
  Source/Swig/scanner.c
  Source/Swig/stype.c
  Source/Swig/symbol.c
  Source/Swig/tree.c
  Source/Swig/typemap.c
  Source/Swig/typeobj.c
  Source/Swig/typesys.c
  Source/Swig/wrapfunc.c
)

add_dependencies(swig gen-parse)

target_sources(swig
  PRIVATE
    ${PROJECT_BINARY_DIR}/parser.tab.h
    ${PROJECT_BINARY_DIR}/parser.tab.c
)

set_target_properties(swig
  PROPERTIES
    PDB_NAME swig
    PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
)
target_include_directories(swig
  PRIVATE
    Source/CParse
    Source/DOH
    Source/Doxygen
    Source/Include
    Source/Modules
    Source/Preprocessor
    Source/Swig
    ${PROJECT_BINARY_DIR}
)

target_compile_definitions(swig
  PRIVATE
  _CRT_SECURE_NO_WARNINGS
  _CRT_STDC_NO_DEPRECATE
)

target_compile_options(swig
  PRIVATE
    /sdl-
)

target_link_libraries(swig
  PRIVATE
    PCRE::pcre
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS swig EXPORT swig
  RUNTIME DESTINATION bin/swig
)

install(DIRECTORY
  Lib
  DESTINATION bin/swig
)

install(EXPORT swig
  NAMESPACE
    ${PROJECT_NAME}::
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)

install(FILES
  cmake/${PROJECT_NAME}-config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)