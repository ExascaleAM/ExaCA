cmake_minimum_required(VERSION 3.12)

project(ExaCA LANGUAGES CXX VERSION 1.1.0)

include(GNUInstallDirs)

find_package(Kokkos 3.2 REQUIRED)
find_package(MPI REQUIRED)

option(ExaCA_ENABLE_JSON "Enable JSON input file support." OFF)
if(ExaCA_ENABLE_JSON)
  find_package(nlohmann_json 3.10.0 QUIET)
  if(NOT NLOHMANN_JSON_FOUND)
    include(FetchContent)
    # Using most recent release here
    FetchContent_Declare(json URL https://github.com/nlohmann/json/releases/download/v3.11.2/json.tar.xz)
    FetchContent_MakeAvailable(json)
  endif()
endif()

find_package(Git)
if(GIT_FOUND AND IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/.git)
  EXECUTE_PROCESS(
    COMMAND           ${GIT_EXECUTABLE} log --pretty=format:%H -n 1
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    OUTPUT_VARIABLE   ExaCA_GIT_COMMIT_HASH
    )
else()
  set(ExaCA_GIT_COMMIT_HASH "Not a git repository")
endif()
message(STATUS "ExaCA commit: ${ExaCA_GIT_COMMIT_HASH}")

# NOTE: this order is specifically used in the unit tests.
set(EXACA_SUPPORTED_DEVICES SERIAL PTHREAD OPENMP CUDA HIP)

add_subdirectory(src)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ExaCAconfig.cmakein
  ${CMAKE_CURRENT_BINARY_DIR}/ExaCAconfig.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/ExaCAconfig.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ExaCA)

# Install data files
file(GLOB MATERIALS examples/Materials/*)
file(GLOB SUBSTRATE examples/Substrate/GrainOrientation*.csv)
install(FILES ${MATERIALS}
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/ExaCA)
install(FILES ${SUBSTRATE}
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/ExaCA)

option(ExaCA_ENABLE_TESTING "Build unit tests" OFF)

if(ExaCA_ENABLE_TESTING)
  find_package(GTest 1.10 REQUIRED)
  # Workaround for FindGTest module in CMake older than 3.20
  if(TARGET GTest::gtest)
    set(gtest_target GTest::gtest)
  elseif(TARGET GTest::GTest)
    set(gtest_target GTest::GTest)
  else()
    message(FATAL_ERROR "bug in GTest find module workaround")
  endif()
  enable_testing()

  set(TEST_HARNESS_DIR ${CMAKE_SOURCE_DIR}/unit_test)
  set(EXACA_TEST_DEVICES)
  foreach(_device ${EXACA_SUPPORTED_DEVICES})
    if(Kokkos_ENABLE_${_device})
      # FIXME: enable multiple test backends when source is templated.
      #list(APPEND EXACA_TEST_DEVICES ${_device})
      set(EXACA_TEST_DEVICES ${_device})
    endif()
  endforeach()

  add_subdirectory(unit_test)
endif()

add_subdirectory(analysis)

# clang-format
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
find_package( CLANG_FORMAT 10)
if(CLANG_FORMAT_FOUND)
  file(GLOB_RECURSE FORMAT_SOURCES src/*.[c,h]pp analysis/*.[c,h]pp unit_test/*.[c,h]pp)
  add_custom_target(format
    COMMAND ${CLANG_FORMAT_EXECUTABLE} -i -style=file ${FORMAT_SOURCES}
    DEPENDS ${FORMAT_SOURCES})
endif()
