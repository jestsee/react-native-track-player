cmake_minimum_required(VERSION 3.13)

project(trackplayer)

get_filename_component(TRACKPLAYER_PACKAGE_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../../" ABSOLUTE)

# Allow callers (Gradle) to override where React Native lives.
# If not provided, try to resolve it from common node_modules layouts.
if(NOT DEFINED REACT_NATIVE_DIR)
  # Typical layout:
  #   <app>/node_modules/react-native-track-player/android/src/main/cpp (this file)
  #   <app>/node_modules/react-native (peer dependency)
  get_filename_component(APP_NODE_MODULES_DIR "${TRACKPLAYER_PACKAGE_DIR}/.." ABSOLUTE)
  if(EXISTS "${APP_NODE_MODULES_DIR}/react-native/ReactCommon/jsi/jsi/jsi.cpp")
    set(REACT_NATIVE_DIR "${APP_NODE_MODULES_DIR}/react-native")
  else()
    # Monorepo/hoisted layout: walk upwards and look for node_modules/react-native
    set(_search_dir "${TRACKPLAYER_PACKAGE_DIR}")
    set(_max_hops 10)
    set(_hop 0)
    while(_hop LESS _max_hops)
      if(EXISTS "${_search_dir}/node_modules/react-native/ReactCommon/jsi/jsi/jsi.cpp")
        set(REACT_NATIVE_DIR "${_search_dir}/node_modules/react-native")
        break()
      endif()
      get_filename_component(_parent "${_search_dir}/.." ABSOLUTE)
      if(_parent STREQUAL _search_dir)
        break()
      endif()
      set(_search_dir "${_parent}")
      math(EXPR _hop "${_hop} + 1")
    endwhile()
  endif()
endif()

if(NOT DEFINED REACT_NATIVE_DIR)
  message(FATAL_ERROR "Could not locate React Native. Set -DREACT_NATIVE_DIR=<path-to-node_modules/react-native>.")
endif()

# React Native headers
set(REACT_COMMON_DIR "${REACT_NATIVE_DIR}/ReactCommon")

# Specify path to React Native includes
include_directories(
  ${REACT_COMMON_DIR}
  ${REACT_COMMON_DIR}/jsi
  ${REACT_COMMON_DIR}/callinvoker
  ${REACT_COMMON_DIR}/react/nativemodule/core
)

add_library(trackplayer SHARED
  JsiBindings.cpp
  JsiUtils.cpp
  ${REACT_COMMON_DIR}/jsi/jsi/jsi.cpp
)

find_library(log-lib log)

target_link_libraries(
  trackplayer
  ${log-lib}
  android
  atomic
  m
)
