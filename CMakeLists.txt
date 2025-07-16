cmake_minimum_required(VERSION 3.5)
project(ndt_omp)

add_definitions(-std=c++17)
set(CMAKE_CXX_FLAGS "-std=c++17")

add_compile_options(-march=native)

# pcl 1.7 causes a segfault when it is built with debug mode
set(CMAKE_BUILD_TYPE "RelWithDebInfo")

# ROS2
find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

find_package(PCL 1.7 REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

message(STATUS "PCL_INCLUDE_DIRS:" ${PCL_INCLUDE_DIRS})
message(STATUS "PCL_LIBRARY_DIRS:" ${PCL_LIBRARY_DIRS})
message(STATUS "PCL_DEFINITIONS:" ${PCL_DEFINITIONS})

find_package(OpenMP)

###########
## Build ##
###########

ament_auto_add_library(ndt_omp SHARED
  src/pclomp/voxel_grid_covariance_omp.cpp
  src/pclomp/ndt_omp.cpp
  src/pclomp/gicp_omp.cpp
)

target_link_libraries(ndt_omp ${PCL_LIBRARIES})

if(OpenMP_CXX_FOUND)
  target_link_libraries(ndt_omp OpenMP::OpenMP_CXX)
else()
  message(WARNING "OpenMP not found")
endif()

#############################
## ROS 2 multigrid ndt_omp ##
#############################
ament_auto_add_library(multigrid_ndt_omp SHARED
  src/multigrid_pclomp/multi_voxel_grid_covariance_omp.cpp
  src/multigrid_pclomp/multigrid_ndt_omp.cpp
  src/estimate_covariance/estimate_covariance.cpp
)
target_link_libraries(multigrid_ndt_omp ${PCL_LIBRARIES})
if(OpenMP_CXX_FOUND)
  target_link_libraries(multigrid_ndt_omp OpenMP::OpenMP_CXX)
else()
  message(WARNING "OpenMP not found")
endif()
###################################
## ROS 2 multigrid ndt_omp (end) ##
###################################

ament_auto_package()

set(EXECUTABLES
  "align"
  "regression_test"
  "check_covariance"
)

foreach(executable IN LISTS EXECUTABLES)
  add_executable(${executable} apps/${executable}.cpp)
  add_dependencies(${executable} ndt_omp multigrid_ndt_omp)
  target_link_libraries(${executable} ndt_omp multigrid_ndt_omp ${PCL_LIBRARIES})
endforeach()
