# Dependency checks
find_package(urdfdom QUIET)
if(urdfdom_FOUND)
  if(DART_VERBOSE)
    message(STATUS "Looking for urdfdom - ${urdfdom_headers_VERSION} found")
  endif()
else()
  message(STATUS "Looking for urdfdom - NOT found, please install liburdfdom-dev")
  return()
endif()
if(MSVC)
  set(urdfdom_LIBRARIES optimized urdfdom_sensor      debug urdfdom_sensord
                        optimized urdfdom_model_state debug urdfdom_model_stated
                        optimized urdfdom_model       debug urdfdom_modeld
                        optimized urdfdom_world       debug urdfdom_worldd
                        optimized console_bridge      debug console_bridged)
endif()

# Search all header and source files
file(GLOB hdrs "*.hpp")
file(GLOB srcs "*.cpp")

# Set local target name
set(target_name ${PROJECT_NAME}-io-urdf)
set(component_name io-urdf)

set(DART_IO_BACKWARDCOMPATIBILITY_HPP_IN
  ${CMAKE_CURRENT_SOURCE_DIR}/BackwardCompatibility.hpp.in
)
set(DART_IO_BACKWARDCOMPATIBILITY_HPP_OUT
  ${CMAKE_CURRENT_BINARY_DIR}/BackwardCompatibility.hpp
)
if(DART_VERBOSE)
  message(STATUS ${DART_IO_BACKWARDCOMPATIBILITY_HPP_OUT})
endif()
configure_file(
  ${DART_IO_BACKWARDCOMPATIBILITY_HPP_IN}
  ${DART_IO_BACKWARDCOMPATIBILITY_HPP_OUT}
  @ONLY
)
install(FILES ${DART_IO_BACKWARDCOMPATIBILITY_HPP_OUT}
  DESTINATION include/dart/io/urdf
)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# Add target
dart_add_library(${target_name} ${hdrs} ${srcs})
target_include_directories(
  ${target_name} SYSTEM
  PUBLIC ${urdfdom_INCLUDE_DIRS}
)
target_link_libraries(
  ${target_name}
  dart-io
  ${urdfdom_LIBRARIES}
)

# Component
add_component(${PROJECT_NAME} ${component_name})
add_component_targets(${PROJECT_NAME} ${component_name} ${target_name})
add_component_dependencies(${PROJECT_NAME} ${component_name} io)

# Generate header for this namespace
dart_get_filename_components(header_names "io_urdf headers" ${hdrs})
# TODO: remove below line once the files are completely removed.
list(REMOVE_ITEM header_names "URDFTypes.hpp")
dart_generate_include_header_file(
  "${CMAKE_CURRENT_BINARY_DIR}/urdf.hpp"
  "dart/io/urdf/"
  ${header_names}
)

# Install
install(
  FILES ${hdrs} ${CMAKE_CURRENT_BINARY_DIR}/urdf.hpp
  DESTINATION include/dart/io/urdf
  COMPONENT headers
)
