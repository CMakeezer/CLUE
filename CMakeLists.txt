cmake_minimum_required(VERSION 2.8)
project(CLUE)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "../parrotscmake")

include(cotire)
include(PrintProperties)

set(CMAKE_VERBOSE_MAKEFILE FALSE)
set(ENABLE_COTIRE_BUILD FALSE)
set(ENABLE_INSTALL TRUE)
set(COTIRE_DEBUG FALSE)
set(COTIRE_VERBOSE FALSE)
set(ENABLE_TEST TRUE)
set(PRINT_TARGET_PROPERTIES FALSE)
set(ENABLE_COVERAGE FALSE)
option(BUILD_TESTS "" TRUE)

#Enable Unit Test coverage
if (ENABLE_COVERAGE)
    APPEND_COVERAGE_COMPILER_FLAGS()
endif ()

## Compiler settings

message(STATUS "CMAKE_SYSTEM_NAME: " ${CMAKE_SYSTEM_NAME})
message(STATUS "CXX_COMPILER_ID: " ${CMAKE_CXX_COMPILER_ID})

set(CMAKE_CXX_STANDARD 11)

if (${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")
    if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
        set(NEWCXX_ENABLED_FLAG "-stdlib=libc++ -Wall -O2")
    else()
        set(NEWCXX_ENABLED_FLAG "-Wall -Wno-mismatched-tags -O2")
    endif()
elseif (${CMAKE_CXX_COMPILER_ID} MATCHES "GNU")
    set(NEWCXX_ENABLED_FLAG "-Wall -O2")
else()
    message(FATAL_ERROR, "Only GCC and clang is supported for now.")
endif()

set(CMAKE_CXX_FLAGS ${NEWCXX_ENABLED_FLAG})
message(STATUS "CMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")

<<<<<<< HEAD
if(MSVC)
    set(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG")
    set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1")
endif()
=======
set(CMAKE_EXE_LINKER_FLAGS  "-Wl,--no-as-needed ${CMAKE_EXE_LINKER_FLAGS}")
>>>>>>> master

enable_testing()

###################################
#
#  Dependencies
#
###################################

## Google Test

find_package(GTest 1.80 REQUIRED)
find_package(Threads REQUIRED)

###################################
#
#  Directories
#
###################################

add_library(clue INTERFACE)
target_include_directories(clue INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        $<INSTALL_INTERFACE:include>)

if (PRINT_TARGET_PROPERTIES)
    print_target_properties(clue)
endif ()

###################################
#
#  Unit testing
#
###################################

set(UNIT_TESTS
    test_misc
    test_predicates
    test_optional
    test_string_view
    test_stringex
    test_mparser
    test_sformat
    test_stemplate
    test_type_name
    test_value_range
    test_array_view
    test_reindexed_view
    test_fast_vector
    test_ordered_dict
    test_keyed_vector
    test_meta
    test_meta_seq
    test_textio
    #test_include_all
    test_shared_mutex
    #test_concurrent_counter
    test_concurrent_queue
    #test_thread_pool
)

if(BUILD_TESTS)
    foreach (tname ${UNIT_TESTS})
        add_executable(${tname} ${CMAKE_CURRENT_SOURCE_DIR}/tests/${tname}.cpp)
        list(APPEND UNIT_TESTS_EXECUTABLES ${tname})
        add_test(NAME ${tname} COMMAND ${tname})
        target_link_libraries(${tname} PRIVATE Threads::Threads GTest::GTest GTest::Main clue)
    endforeach()
endif()

if (ENABLE_COVERAGE)
    #SETUP_TARGET_FOR_COVERAGE(NAME clue_coverage EXECUTABLE test_misc DEPENDENCIES ${UNIT_TESTS_EXECUTABLES}) TODO: EXECUTABLE ahould invoke all depended test executables
endif ()

###################################
#
#  Examples
#
###################################

set(EXAMPLES
    ex_stopwatch
    ex_timing
    ex_strings
    ex_mparser
)

set(THREAD_EXAMPLES
    ex_cccounter
    ex_ccqueue
    #ex_threadpool
)

foreach (name ${EXAMPLES})
    add_executable(${name} examples/${name}.cpp)
    target_link_libraries(${name} PRIVATE clue)
endforeach()

foreach (name ${THREAD_EXAMPLES})
    add_executable(${name} examples/${name}.cpp)
    target_link_libraries(${name} ${CMAKE_THREAD_LIBS_INIT} clue)
endforeach()


###INSTALL
set(include_install_dir "include")
set(lib_install_dir "lib/")
set(bin_install_dir "bin/")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(pkgconfig_install_dir "lib/pkgconfig")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(pkg_config "${generated_dir}/${PROJECT_NAME}.pc")
set(targets_export_name "${PROJECT_NAME}Targets")

set(namespace "${PROJECT_NAME}::")

set(PROJECT_VERSION "1.0.0")
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        "${version_config}" VERSION ${GIT_VERSION} COMPATIBILITY SameMajorVersion
)

configure_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in" "${project_config}" @ONLY)
configure_file("${PROJECT_SOURCE_DIR}/cmake/clue.pc.in" "${pkg_config}" @ONLY)

if (ENABLE_INSTALL)
    # Install target (will install the library to specified CMAKE_INSTALL_PREFIX variable)
    install(
            TARGETS clue
            EXPORT ${targets_export_name}
            ARCHIVE DESTINATION ${lib_install_dir}
            LIBRARY DESTINATION ${lib_install_dir}
            RUNTIME DESTINATION ${bin_install_dir}
            INCLUDES DESTINATION ${include_install_dir})

    install(
            DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
            DESTINATION ${include_install_dir}
            FILES_MATCHING PATTERN "*.*hpp")

    install(
            FILES "${project_config}" "${version_config}"
            DESTINATION "${config_install_dir}")

    install(
            FILES "${pkg_config}"
            DESTINATION "${pkgconfig_install_dir}")

    install(
            EXPORT "${targets_export_name}"
            NAMESPACE "${namespace}"
            DESTINATION "${config_install_dir}")
endif ()
