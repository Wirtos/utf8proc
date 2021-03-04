cmake_minimum_required(VERSION 3.10)

include(utils.cmake)

disallow_intree_builds()

project(utf8proc C)

# This is the ABI version number, which may differ from the
# API version number (defined in utf8proc.h).
# Be sure to also update these in Makefile and MANIFEST!
set(SO_MAJOR 2)
set(SO_MINOR 4)
set(SO_PATCH 1)

option(UTF8PROC_INSTALL "Enable installation of utf8proc" On)
option(UTF8PROC_ENABLE_TESTING "Enable testing of utf8proc" Off)
option(LIB_FUZZING_ENGINE "Fuzzing engine to link against" Off)
option(BUILD_SHARED_LIBS "Whether to build shared or dynamic library" Off)

add_library(utf8proc
        utf8proc.c
        utf8proc.h
        )

include(GenerateExportHeader)
generate_export_header(${PROJECT_NAME})

if (${CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT})
    set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/install" CACHE PATH "" FORCE)
endif ()

include(GNUInstallDirs)

target_include_directories(${PROJECT_NAME} PUBLIC
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)


set_property(TARGET ${PROJECT_NAME}
        PROPERTY PUBLIC_HEADER
        utf8proc.h
        "${CMAKE_CURRENT_BINARY_DIR}/utf8proc_export.h")

set_target_properties(${PROJECT_NAME} PROPERTIES
        PREFIX "lib"
        POSITION_INDEPENDENT_CODE ON
        VERSION "${SO_MAJOR}.${SO_MINOR}.${SO_PATCH}"
        SOVERSION ${SO_MAJOR})

install(TARGETS
        ${PROJECT_NAME}
        EXPORT UTF8ProcConfig
        )

export(TARGETS
        ${PROJECT_NAME}
        FILE "${CMAKE_CURRENT_BINARY_DIR}/UTF8ProcConfig.cmake"
        )

install(EXPORT
        UTF8ProcConfig
        DESTINATION "${CMAKE_INSTALL_DATADIR}/cmake"
        )


if (1)
    enable_testing()
    file(MAKE_DIRECTORY data)
    set(UNICODE_VERSION 13.0.0)
    file(DOWNLOAD https://www.unicode.org/Public/${UNICODE_VERSION}/ucd/NormalizationTest.txt ${CMAKE_BINARY_DIR}/data/NormalizationTest.txt SHOW_PROGRESS)
    file(DOWNLOAD https://www.unicode.org/Public/${UNICODE_VERSION}/ucd/auxiliary/GraphemeBreakTest.txt ${CMAKE_BINARY_DIR}/data/GraphemeBreakTest.txt SHOW_PROGRESS)
    add_executable(case test/tests.h test/tests.c utf8proc.h test/case.c)
    target_link_libraries(case utf8proc)
    add_executable(custom test/tests.h test/tests.c utf8proc.h test/custom.c)
    target_link_libraries(custom utf8proc)
    add_executable(iterate test/tests.h test/tests.c utf8proc.h test/iterate.c)
    target_link_libraries(iterate utf8proc)
    add_executable(misc test/tests.h test/tests.c utf8proc.h test/misc.c)
    target_link_libraries(misc utf8proc)
    add_executable(printproperty test/tests.h test/tests.c utf8proc.h test/printproperty.c)
    target_link_libraries(printproperty utf8proc)
    add_executable(valid test/tests.h test/tests.c utf8proc.h test/valid.c)
    target_link_libraries(valid utf8proc)
    add_test(utf8proc.testcase case)
    add_test(utf8proc.testcustom custom)
    add_test(utf8proc.testiterate iterate)
    add_test(utf8proc.testmisc misc)
    add_test(utf8proc.testprintproperty printproperty)
    add_test(utf8proc.testvalid valid)

    if (NOT WIN32)
        # no wcwidth function on Windows
        add_executable(charwidth test/tests.h test/tests.c utf8proc.h test/charwidth.c)
        target_link_libraries(charwidth utf8proc)
        add_test(utf8proc.testcharwidth charwidth)
    endif ()
    add_executable(graphemetest test/tests.h test/tests.c utf8proc.h test/graphemetest.c)
    target_link_libraries(graphemetest utf8proc)
    add_executable(normtest test/tests.h test/tests.c utf8proc.h test/normtest.c)
    target_link_libraries(normtest utf8proc)
    add_test(utf8proc.testgraphemetest graphemetest data/GraphemeBreakTest.txt)
    add_test(utf8proc.testnormtest normtest data/NormalizationTest.txt)

    if (LIB_FUZZING_ENGINE)
        add_executable(fuzzer utf8proc.h test/fuzzer.c)
        target_link_libraries(fuzzer ${LIB_FUZZING_ENGINE} utf8proc)
    else ()
        add_executable(fuzzer utf8proc.h test/fuzz_main.c test/fuzzer.c)
        target_link_libraries(fuzzer utf8proc)
    endif ()
endif ()
