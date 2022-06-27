project(amcl3d)
cmake_minimum_required(VERSION 3.5.1)

add_compile_options(-std=c++14)

##################################
## Configure CATKIN dependecies ##
##################################

# Mod by Tim:
#find_package(PCL REQUIRED COMPONENTS core features filters io segmentation surface)
set(PCL_FOLDER /media/Data/workspace/decada_robotics_ws/test_folder/pcl/pcl-pcl-1.9.1/build/install)
find_package(PCL 1.9 REQUIRED PATHS ${PCL_FOLDER} NO_DEFAULT_PATH)

find_package(catkin REQUIRED
  COMPONENTS
    visualization_msgs
    nav_msgs
    octomap_ros
    pcl_ros
    rosinrange_msg
)

###################################
## CATKIN specific configuration ##
###################################

catkin_package(
  CATKIN_DEPENDS
    visualization_msgs
    nav_msgs
    octomap_ros
    pcl_ros
    rosinrange_msg
)

#######################
## Configuring GMock ##
#######################

# find_package(GMock)

###########
## Build ##
###########

include_directories(SYSTEM ${catkin_INCLUDE_DIRS})

add_library(${PROJECT_NAME}
  src/Grid3d.cpp
  src/Grid3d.h
  src/ParticleFilter.cpp
  src/ParticleFilter.h
  src/PointCloudTools.cpp
  src/PointCloudTools.h)
add_dependencies(${PROJECT_NAME}      ${catkin_EXPORTED_TARGETS})
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})

add_executable(${PROJECT_NAME}_node
  src/main.cpp
  src/Node.cpp
  src/Node.h
  src/Parameters.cpp
  src/Parameters.h)
add_dependencies(${PROJECT_NAME}_node       ${catkin_EXPORTED_TARGETS})
target_link_libraries(${PROJECT_NAME}_node  ${catkin_LIBRARIES} ${PROJECT_NAME} ${PCL_libraries})

install(TARGETS ${PROJECT_NAME}
  ${PROJECT_NAME}_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

#install(DIRECTORY include/${PROJECT_NAME}/
#  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)

#############
## Testing ##
#############

if(GMOCK_FOUND)
  include_directories(src)
  include_directories(SYSTEM ${GMOCK_INCLUDE_DIRS})

  file(GLOB_RECURSE TESTS_SRCS  "tests/*.c*")
  file(GLOB_RECURSE TESTS_HDRS  "tests/*.h*")

  catkin_add_gtest(${PROJECT_NAME}_tests
    ${TESTS_SRCS} ${TESTS_HDRS})
  if(TARGET ${PROJECT_NAME}_tests)
    target_compile_definitions(${PROJECT_NAME}_tests PRIVATE
      PROJECT_SOURCE_DIR="${PROJECT_SOURCE_DIR}")
    target_link_libraries(${PROJECT_NAME}_tests
      ${GMOCK_BOTH_LIBRARIES} ${PROJECT_NAME})
  endif()
endif()
