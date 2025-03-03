cmake_minimum_required(VERSION 3.14)

set(LIBRARY_NAME "flecs_components_geometry")
set(LIBRARY_ALIAS "flecs_hub::components::geometry")

project(${LIBRARY_NAME} VERSION 1.0.0 LANGUAGES C)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there.")
endif()

##################################
########### LIBRARY ##############
##################################

file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")
add_library(${LIBRARY_NAME} ${HEADERS} ${SOURCES})
add_library(${LIBRARY_ALIAS} ALIAS ${LIBRARY_NAME})
target_include_directories(${LIBRARY_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)
set_target_properties(${LIBRARY_NAME} PROPERTIES C_STANDARD 99)

target_compile_definitions(
  # TARGET_PROPERTY:TYPE == STATIC_LIBRARY ? ${LIBRARY_NAME}_STATIC : ""
  ${LIBRARY_NAME} PUBLIC $<$<STREQUAL:$<TARGET_PROPERTY:${LIBRARY_NAME},TYPE>,STATIC_LIBRARY>:;${LIBRARY_NAME}_STATIC>
)

##################################
########## DEPENDENCIES ##########
##################################

get_target_property(BUILD_LIBRARY_TYPE ${LIBRARY_NAME} TYPE)
if(BUILD_LIBRARY_TYPE STREQUAL "STATIC_LIBRARY")
  set(STR_IS_STATIC_LIBRARY "ON")
  set(STR_IS_SHARED_LIBRARY "OFF")
else()
  set(STR_IS_STATIC_LIBRARY "OFF")
  set(STR_IS_SHARED_LIBRARY "ON") 
endif()

include(cmake/get_cpm.cmake)

CPMAddPackage(
  VERSION 3.2.8
  GITHUB_REPOSITORY SanderMertens/flecs
  OPTIONS "FLECS_STATIC ${STR_IS_STATIC_LIBRARY}"
          "FLECS_SHARED ${STR_IS_SHARED_LIBRARY}"
)
target_link_libraries(${LIBRARY_NAME} PUBLIC
  # TARGET_PROPERTY:TYPE == STATIC_LIBRARY ? flecs::flecs_static : flecs::flecs
  $<IF:$<STREQUAL:$<TARGET_PROPERTY:${LIBRARY_NAME},TYPE>,STATIC_LIBRARY>,flecs::flecs_static,flecs::flecs>
)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-graphics#4a8722f770ab1ea300935da9a7043f9d826870e6")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::graphics)