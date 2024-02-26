cmake_minimum_required(VERSION 3.10)

# Read src/cmock.h file and get project version from it
set(CMOCK_HEADER "src/cmock.h")

file(STRINGS "${CMOCK_HEADER}" CMOCK_HEADER_CONTENT
    REGEX "^#define CMOCK_VERSION_(MAJOR|MINOR|BUILD) +[0-9]+$"
)

set(CMOCK_HEADER_VERSION_MAJOR 0)
set(CMOCK_HEADER_VERSION_MINOR 0)
set(CMOCK_HEADER_VERSION_BUILD 0)

foreach(VERSION_LINE IN LISTS CMOCK_HEADER_CONTENT)
    foreach(VERSION_PART MAJOR MINOR BUILD)
        string(CONCAT REGEX_STRING "#define CMOCK_VERSION_"
                                   "${VERSION_PART}"
                                   " +([0-9]+)"
        )

        if(VERSION_LINE MATCHES "${REGEX_STRING}")
            set(CMOCK_HEADER_VERSION_${VERSION_PART} "${CMAKE_MATCH_1}")
        endif()
    endforeach()
endforeach()

project(cmock
    VERSION     ${CMOCK_HEADER_VERSION_MAJOR}.${CMOCK_HEADER_VERSION_MINOR}.${CMOCK_HEADER_VERSION_BUILD}
    LANGUAGES   C
    DESCRIPTION "C Unit testing mocking framework."
)

# Main target ------------------------------------------------------------------
add_library(${PROJECT_NAME} STATIC)
add_library(${PROJECT_NAME}::framework ALIAS ${PROJECT_NAME})

# Includes ---------------------------------------------------------------------
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)


target_sources(${PROJECT_NAME}
    PRIVATE
        src/cmock.c
)

target_link_libraries(${PROJECT_NAME} unity)

target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
)

set(${PROJECT_NAME}_PUBLIC_HEADERS
        src/cmock.h
        src/cmock_internals.h
)

set_target_properties(${PROJECT_NAME}
    PROPERTIES
        C_STANDARD          11
        C_STANDARD_REQUIRED ON
        C_EXTENSIONS        OFF
        PUBLIC_HEADER       "${${PROJECT_NAME}_PUBLIC_HEADERS}"
        EXPORT_NAME         framework
)


write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake
    VERSION       ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

## Target installation
install(TARGETS   ${PROJECT_NAME}
    EXPORT        ${PROJECT_NAME}Targets
    ARCHIVE       DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY       DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    COMPONENT     library
)

## Target's cmake files: targets export
install(EXPORT  ${PROJECT_NAME}Targets
    NAMESPACE   ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

## Target's cmake files: config and version config for find_package()
install(FILES   ${PROJECT_NAME}Config.cmake
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)