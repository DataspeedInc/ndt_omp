cmake_minimum_required(VERSION 3.5)
project(ndt_omp)

# should use march=native ?
# add_definitions(-std=c++14 -msse -msse2 -msse3 -msse4 -msse4.1 -msse4.2)
# set(CMAKE_CXX_FLAGS "-std=c++14 -msse -msse2 -msse3 -msse4 -msse4.1 -msse4.2")

# pcl 1.7 causes a segfault when it is built with debug mode
# set(CMAKE_BUILD_TYPE "RELEASE")
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# find_package(catkin REQUIRED COMPONENTS
#   roscpp
#   pcl_ros
# )
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(pcl_ros REQUIRED)
find_package(Eigen3 REQUIRED)

find_package(PCL 1.7 REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

message(STATUS "PCL_INCLUDE_DIRS:" ${PCL_INCLUDE_DIRS})
message(STATUS "PCL_LIBRARY_DIRS:" ${PCL_LIBRARY_DIRS})
message(STATUS "PCL_DEFINITIONS:" ${PCL_DEFINITIONS})

find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

add_library(${PROJECT_NAME} SHARED
  src/pclomp/voxel_grid_covariance_omp.cpp
  src/pclomp/ndt_omp.cpp
  src/pclomp/gicp_omp.cpp
)
target_include_directories(
  ${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(${PROJECT_NAME}
  ${PCL_LIBRARIES}
)
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_dependencies(rclcpp)
ament_export_dependencies(pcl_ros)

add_executable(align
  apps/align.cpp
)
ament_target_dependencies(align
  rclcpp
)
target_link_libraries(align
  ${PCL_LIBRARIES}
  ndt_omp
)

install(
  TARGETS
    ${PROJECT_NAME}
    align
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY include/
  DESTINATION include
)

ament_package()

###################################
## catkin specific configuration ##
###################################
# catkin_package(
#   INCLUDE_DIRS include
#   LIBRARIES ndt_omp
# )

###########
## Build ##
###########
# include_directories(include)
# include_directories(
#   ${catkin_INCLUDE_DIRS}
# )

# add_library(ndt_omp
#   src/pclomp/voxel_grid_covariance_omp.cpp
#   src/pclomp/ndt_omp.cpp
#   src/pclomp/gicp_omp.cpp
# )

# add_executable(align
#   apps/align.cpp
# )
# add_dependencies(align
#   ndt_omp
# )
# target_link_libraries(align
#   ${catkin_LIBRARIES}
#   ${PCL_LIBRARIES}
#   ndt_omp
# )

#############
## INSTALL ##
#############

# install(
#   TARGETS
#     ndt_omp
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# )

# install headers
# install(DIRECTORY include/pclomp
#         DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION})
