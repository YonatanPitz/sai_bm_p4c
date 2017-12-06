# Copyright 2013-present Barefoot Networks, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Makefile for a backend that generates code for the Behavioral Model version 2 (BMv2)
# compiling for the simple_switch target.

# sources for backend executable
set (BMV2_SRCS bmv2.cpp)

set (BMV2_BACKEND_SRCS
  analyzer.cpp
  action.cpp
  backend.cpp
  control.cpp
  deparser.cpp
  errorcode.cpp
  expression.cpp
  extern.cpp
  header.cpp
  helpers.cpp
  extractArchInfo.cpp
  JsonObjects.cpp
  lower.cpp
  midend.cpp
  metermap.cpp
  parser.cpp
  portableSwitch.cpp
  sharedActionSelectorCheck.cpp
  simpleSwitch.cpp
  synthesizeValidField.cpp
  )

set (BMV2_BACKEND_HDRS
  analyzer.h
  action.h
  backend.h
  control.h
  deparser.h
  errorcode.h
  expression.h
  extern.h
  header.h
  helpers.h
  extractArchInfo.h
  JsonObjects.h
  lower.h
  midend.h
  metermap.h
  options.h
  parser.h
  portableSwitch.h
  sharedActionSelectorCheck.h
  simpleSwitch.h
  synthesizeValidField.h
  )

#set (IR_DEF_FILES ${IR_DEF_FILES} ${CMAKE_CURRENT_SOURCE_DIR}/bmv2.def PARENT_SCOPE)

add_library(bmv2backendflx ${BMV2_BACKEND_SRCS})
add_dependencies(bmv2backendflx genIR)


# for flextrum (json=>c api) add python support:
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS})

add_executable(p4c-sai-bm ${BMV2_SRCS})
target_link_libraries (p4c-sai-bm bmv2backendflx ${P4C_LIBRARIES} ${P4C_LIB_DEPS} ${PYTHON_LIBRARIES})

install(TARGETS p4c-sai-bm RUNTIME DESTINATION ${P4C_RUNTIME_OUTPUT_DIRECTORY})
install (DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/p4include
   DESTINATION ${P4C_ARTIFACTS_OUTPUT_DIRECTORY})
install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/driver/p4c.p4c-sai-bm.cfg
  DESTINATION ${P4C_ARTIFACTS_OUTPUT_DIRECTORY}/p4c_src)

set (BMV2_SOURCES ${BMV2_BACKEND_SRCS} ${BMV2_SRCS} ${BMV2_BACKEND_HDRS})
add_cpplint_files (${CMAKE_CURRENT_SOURCE_DIR} "${BMV2_SOURCES}")

# hack to get around the fact that the test scripts expect the backend
# binary to be in the top level directory. This should go away when we
# remove automake and fix the scripts.
add_custom_target(linksai
  COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_CURRENT_BINARY_DIR}/p4c-sai-bm ${P4C_BINARY_DIR}/p4c-sai-bm
  COMMAND ${CMAKE_COMMAND} -E create_symlink ${P4C_BINARY_DIR}/p4include ${CMAKE_CURRENT_BINARY_DIR}/p4include
  COMMAND ${CMAKE_COMMAND} -E create_symlink ${P4C_BINARY_DIR}/p4_14include ${CMAKE_CURRENT_BINARY_DIR}/p4_14include
  )
add_dependencies(p4c_driver linksai)

