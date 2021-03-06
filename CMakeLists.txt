cmake_minimum_required(VERSION 3.8.0 FATAL_ERROR)

project(taopq VERSION 1.0.0 LANGUAGES CXX)

if(EXISTS "${CMAKE_BINARY_DIR}/conanbuildinfo.cmake")
  include("${CMAKE_BINARY_DIR}/conanbuildinfo.cmake")
  conan_basic_setup()
endif()

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake)

find_package(PostgreSQL REQUIRED)

set(TAOPQ_INSTALL_INCLUDE_DIR "include" CACHE STRING "The installation include directory")
set(TAOPQ_INSTALL_DOC_DIR "share/doc/tao/pq" CACHE STRING "The installation doc directory")
option(TAOPQ_BUILD_TESTS "Build test programs" ON)

set(TAOPQ_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/include)

set(TAOPQ_INCLUDE_FILES
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/table_writer.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/connection_pool.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/null.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/transaction.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/field.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/result.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/row.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/result_traits.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/result_traits_tuple.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/result_traits_optional.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/parameter_traits.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/result_traits_pair.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/connection.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/internal/strtox.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/internal/demangle.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/internal/printf.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq/internal/pool.hpp
  ${TAOPQ_INCLUDE_DIRS}/tao/pq.hpp
)

set(TAOPQ_SOURCE_FILES
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/transaction.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/result.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/row.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/connection.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/table_writer.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/connection_pool.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/result_traits.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/field.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/internal/strtox.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/internal/printf.cpp
  ${CMAKE_CURRENT_LIST_DIR}/src/lib/pq/internal/demangle.cpp
)

source_group("Header Files" FILES ${TAOPQ_INCLUDE_FILES})

add_library(taopq ${TAOPQ_SOURCE_FILES} ${TAOPQ_INCLUDE_FILES})
add_library(taocpp::taopq ALIAS taopq)
set_target_properties(taopq PROPERTIES OUTPUT_NAME "${PROJECT_NAME}")

target_include_directories(taopq
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${PostgreSQL_INCLUDE_DIRS}
)

target_link_libraries(taopq PUBLIC ${PostgreSQL_LIBRARIES})

target_compile_features(taopq PUBLIC cxx_std_17)

include(GNUInstallDirs)
set(TAOPQ_INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME} CACHE STRING "The installation cmake directory")

install(TARGETS taopq EXPORT taopq-targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY include/ DESTINATION ${TAOPQ_INSTALL_INCLUDE_DIR})
install(FILES LICENSE DESTINATION ${TAOPQ_INSTALL_DOC_DIR})

install(EXPORT taopq-targets
  FILE ${PROJECT_NAME}Targets.cmake
  NAMESPACE taocpp::
  DESTINATION ${TAOPQ_INSTALL_CONFIGDIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${TAOPQ_INSTALL_CONFIGDIR}
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION ${TAOPQ_INSTALL_CONFIGDIR}
)

export(EXPORT taopq-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake NAMESPACE taocpp::)
export(PACKAGE taopq)

enable_testing()
if(TAOPQ_BUILD_TESTS)
  add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/src/test/pq)
endif()
