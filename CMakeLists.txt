cmake_minimum_required(VERSION 3.1)
project(qp_solver_collection)

add_compile_options(-std=c++17)

# Options for solvers
option(DEFAULT_ENABLE_ALL "Enable all QP solvers by default" OFF)
set(DEFAULT_ENABLE_VALUE ${DEFAULT_ENABLE_ALL})

option(ENABLE_QLD "Enable QLD" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_QUADPROG "Enable QuadProg" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_LSSOL "Enable LSSOL" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_JRLQP "Enable JRLQP" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_QPOASES "Enable qpOASES" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_OSQP "Enable OSQP" ${DEFAULT_ENABLE_VALUE})
option(ENABLE_NASOQ "Enable NASOQ" ${DEFAULT_ENABLE_VALUE})

option(SKIP_PRIVATE_SOLVER_TEST "Skip private solver test" OFF)
if(${SKIP_PRIVATE_SOLVER_TEST})
  add_definitions(-DSKIP_PRIVATE_SOLVER_TEST)
endif()

find_package(catkin REQUIRED COMPONENTS
  roscpp
  )

# Eigen
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})

set(EXIST_ENABLED_SOLVER FALSE)
macro(add_qp_solver _SOLVER_NAME _PACKAGE_NAME)
  if(${ENABLE_${_SOLVER_NAME}})
    if(NOT "${_PACKAGE_NAME}" STREQUAL "")
      find_package(${_PACKAGE_NAME} REQUIRED)
    endif()
    set(EXIST_ENABLED_SOLVER TRUE)
  endif()
  message(NOTICE "- ENABLE_${_SOLVER_NAME}: ${ENABLE_${_SOLVER_NAME}}")
endmacro()

# eigen-qld
add_qp_solver(QLD eigen-qld)

# eigen-quadprog
add_qp_solver(QUADPROG eigen-quadprog)

# eigen-lssol
add_qp_solver(LSSOL eigen-lssol)

# JRLQP
add_qp_solver(JRLQP jrl-qp)

# qpOASES
add_qp_solver(QPOASES "")

# OsqpEigen
add_qp_solver(OSQP OsqpEigen)

# NASOQ
add_qp_solver(NASOQ nasoq)

if(NOT ${EXIST_ENABLED_SOLVER})
  message(FATAL_ERROR "None of the QP solvers are enabled. Enable at least one QP solver.")
endif()

catkin_package(
  CATKIN_DEPENDS
  roscpp
  DEPENDS EIGEN3
  INCLUDE_DIRS include
  LIBRARIES QpSolverCollection
  CFG_EXTRAS qp_solver_options.cmake
  )

set(QP_SOLVER_OPTIONS_HEADER_FILE "${PROJECT_SOURCE_DIR}/include/qp_solver_collection/QpSolverOptions.h")
configure_file("${QP_SOLVER_OPTIONS_HEADER_FILE}.in" "${QP_SOLVER_OPTIONS_HEADER_FILE}")

set(QP_SOLVER_OPTIONS_CMAKE_FILE "${PROJECT_SOURCE_DIR}/cmake/qp_solver_options.cmake")
configure_file("${QP_SOLVER_OPTIONS_CMAKE_FILE}.in" "${QP_SOLVER_OPTIONS_CMAKE_FILE}")

add_subdirectory(src)

if(CATKIN_ENABLE_TESTING)
  add_subdirectory(tests)
endif()

OPTION(INSTALL_DOCUMENTATION "Generate and install the documentation" OFF)
if(INSTALL_DOCUMENTATION)
  add_subdirectory(doc)
endif()
