cmake_minimum_required(VERSION 3.10)
project(trlc VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Installation destinations
if(UNIX OR VXWORKS)
    include(GNUInstallDirs)
    set(TRLC_INC_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")
    set(TRLC_RUNTIME_INSTALL_DIR "${CMAKE_INSTALL_BINDIR}/${PROJECT_NAME}")
    set(TRLC_LIBRARY_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}")
    set(TRLC_ARCHIVE_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}")
    set(TRLC_FRAMEWORK_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}")
    set(TRLC_CMAKE_CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
    set(TRLC_ADDITIONAL_FILES_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}")
elseif(WIN32)
    set(TRLC_INC_INSTALL_DIR "include/${PROJECT_NAME}")
    set(TRLC_RUNTIME_INSTALL_DIR "bin/${PROJECT_NAME}")
    set(TRLC_LIBRARY_INSTALL_DIR "bin/${PROJECT_NAME}")
    set(TRLC_ARCHIVE_INSTALL_DIR "lib/${PROJECT_NAME}")
    set(TRLC_FRAMEWORK_INSTALL_DIR "bin/${PROJECT_NAME}")
    set(TRLC_CMAKE_CONFIG_INSTALL_DIR "share/cmake/${PROJECT_NAME}")
    set(TRLC_ADDITIONAL_FILES_INSTALL_DIR "share/${PROJECT_NAME}")
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${PROJECT_NAME}")
else()
    message(FATAL_ERROR "Could not set install folders for this platform!")
endif()


# Doxygen documentation
find_package(Doxygen)

if(DOXYGEN_FOUND)
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/docs)

    add_custom_target(doc
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_IN}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM
    )
endif()

set(TRLC_BUILD_TESTS ON)
set(TRLC_BUILD_EXAMPLE ON)

if(TRLC_BUILD_TESTS)
    enable_testing()
endif()

# Add sublibraries
add_subdirectory(common)
add_subdirectory(thread_safe)

# Export the package for find_package
include(CMakePackageConfigHelpers)

# Install and export targets
install(EXPORT trlcTargets
  NAMESPACE trlc::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/trlc
)

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/trlcConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/trlcConfig.cmake
  INSTALL_DESTINATION ${TRLC_CMAKE_CONFIG_INSTALL_DIR}
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/trlcConfig.cmake
  DESTINATION ${TRLC_CMAKE_CONFIG_INSTALL_DIR}
)