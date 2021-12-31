cmake_minimum_required(VERSION 3.8)

project(cli-cmake LANGUAGES CXX)

function(cliMain)
    option(CLI_BuildExamples "Build the examples." OFF)
    option(CLI_BuildTests "Build the unit tests." OFF)
    option(CLI_UseBoostAsio "Use the boost asio library." ON)
    option(CLI_UseStandaloneAsio "Use the standalone asio library." OFF)

    if(WIN32)
        macro(get_WIN32_WINNT version)
            if(CMAKE_SYSTEM_VERSION)
                set(ver ${CMAKE_SYSTEM_VERSION})
                string(REGEX MATCH "^([0-9]+).([0-9])" ver ${ver})
                string(REGEX MATCH "^([0-9]+)" verMajor ${ver})
                # Check for Windows 10, b/c we'll need to convert to hex 'A'.
                if("${verMajor}" MATCHES "10")
                    set(verMajor "A")
                    string(REGEX REPLACE "^([0-9]+)" ${verMajor} ver ${ver})
                endif()
                # Remove all remaining '.' characters.
                string(REPLACE "." "" ver ${ver})
                # Prepend each digit with a zero.
                string(REGEX REPLACE "([0-9A-Z])" "0\\1" ver ${ver})
                set(${version} "0x${ver}")
            endif()
        endmacro()

        get_WIN32_WINNT(ver)
        add_definitions(-D_WIN32_WINNT=${ver})
    endif()

    if (CLI_UseStandaloneAsio)
        find_path(STANDALONE_ASIO_INCLUDE_PATH NAMES "asio.hpp" HINTS ${ASIO_INCLUDEDIR})
        mark_as_advanced(STANDALONE_ASIO_INCLUDE_PATH)
    endif()

    find_package(Threads REQUIRED)

    # Add Library
    add_library(thirdparty_cli INTERFACE)

    target_include_directories(thirdparty_cli
        INTERFACE
        ${CMAKE_CURRENT_SOURCE_DIR}/cli/include
        )

    target_link_libraries(thirdparty_cli INTERFACE Threads::Threads)
    if (CLI_UseBoostAsio)
        target_link_libraries(thirdparty_cli INTERFACE thirdparty_boost)
        target_compile_definitions(thirdparty_cli INTERFACE BOOST_ASIO_NO_DEPRECATED=1)
    endif()

    if (CLI_UseStandaloneAsio)
        add_library(standalone_asio INTERFACE IMPORTED)
        target_include_directories(standalone_asio SYSTEM INTERFACE ${STANDALONE_ASIO_INCLUDE_PATH})
        target_link_libraries(standalone_asio INTERFACE Threads::Threads)
        target_link_libraries(thirdparty_cli INTERFACE standalone_asio)

        # alternative way:
        # target_include_directories(cli SYSTEM INTERFACE ${STANDALONE_ASIO_INCLUDE_PATH})
    endif()

    target_compile_features(thirdparty_cli INTERFACE cxx_std_14)

    # Examples
    if (CLI_BuildExamples)
        add_subdirectory(examples)
    endif()

    # Tests
    if (CLI_BuildTests)
        enable_testing()
        add_subdirectory(test)
    endif()

endfunction()

cliMain()
