cmake_minimum_required(VERSION 3.1.3)
project(opengv VERSION 1.0 LANGUAGES CXX)

# Set the build type.  Options are:
#
#  None (CMAKE_C_FLAGS or CMAKE_CXX_FLAGS used)
#  Debug (CMAKE_C_FLAGS_DEBUG or CMAKE_CXX_FLAGS_DEBUG)
#  Release (CMAKE_C_FLAGS_RELEASE or CMAKE_CXX_FLAGS_RELEASE)
#  RelWithDebInfo (CMAKE_C_FLAGS_RELWITHDEBINFO or CMAKE_CXX_FLAGS_RELWITHDEBINFO)
#  MinSizeRel (CMAKE_C_FLAGS_MINSIZEREL or CMAKE_CXX_FLAGS_MINSIZEREL)

set(CMAKE_BUILD_TYPE Release)
#set(CMAKE_BUILD_TYPE Debug)

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)

OPTION(BUILD_TESTS "Build tests" ON)
OPTION(BUILD_PYTHON "Build Python extension" OFF)

IF(MSVC)
  set(BUILD_SHARED_LIBS OFF)
ELSE()
  OPTION(BUILD_SHARED_LIBS "Build shared libraries" OFF)
  OPTION(BUILD_POSITION_INDEPENDENT_CODE "Build position independent code (-fPIC)" ON)
ENDIF()

IF(MSVC)
  add_compile_options(/wd4514 /wd4267 /bigobj)
  add_definitions(-D_USE_MATH_DEFINES)
ELSE()
  IF (CMAKE_SYSTEM_PROCESSOR MATCHES "(arm64)|(ARM64)|(aarch64)|(AARCH64)")
    add_definitions (-march=armv8-a)
  ELSEIF (CMAKE_SYSTEM_PROCESSOR MATCHES 
          "(arm)|(ARM)|(armhf)|(ARMHF)|(armel)|(ARMEL)")
    add_definitions (-march=armv7-a)
  ELSE ()
    add_definitions (-march=native) #TODO use correct c++11 def once everybody has moved to gcc 4.7 # for now I even removed std=gnu++0x
  ENDIF()
  add_definitions (
    -O3
    -Wall
    -Wextra
    #-Werror
    -Wwrite-strings
    -Wno-unused-parameter
    -fno-strict-aliasing
  )
ENDIF()

IF(BUILD_POSITION_INDEPENDENT_CODE)
  add_definitions( -fPIC )
ENDIF()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/modules/")
find_package(Eigen REQUIRED)
set(ADDITIONAL_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIR}/unsupported)

add_subdirectory(opengv_lib)
add_subdirectory(test)


# Configuration
set(config_install_dir "lib/cmake/${PROJECT_NAME}-${PROJECT_VERSION}")
set(include_install_dir "opengv_lib/include")
set(version_config "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion )

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "modules/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}")

# Export '<PROJECT-NAME>Targets.cmake' to build dir (to find package in build dir without install)
export(TARGETS opengv FILE "${CMAKE_CURRENT_BINARY_DIR}/${targets_export_name}.cmake")

# Targets:
install(TARGETS opengv
        EXPORT "${targets_export_name}"
        LIBRARY DESTINATION "lib"
        ARCHIVE DESTINATION "lib"
        RUNTIME DESTINATION "bin"
        INCLUDES DESTINATION "${include_install_dir}")

# Config
#   * <prefix>/lib/cmake/opengv/opengvConfig.cmake
#   * <prefix>/lib/cmake/opengv/opengvConfigVersion.cmake
install(FILES "${project_config}" "${version_config}"
        DESTINATION "${config_install_dir}")

# Config
#   * <prefix>/lib/cmake/opengv/opengvTargets.cmake
install(EXPORT "${targets_export_name}"
        NAMESPACE "${namespace}"
        DESTINATION "${config_install_dir}")

# Headers
install(DIRECTORY opengv_lib/include/
        DESTINATION ${include_install_dir} 
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")


if (BUILD_PYTHON)
  add_subdirectory( python )
endif()
