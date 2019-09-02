cmake_minimum_required(VERSION 2.8.3)
project(pic_service)

set( CMAKE_VERBOSE_MAKEFILE true)
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  sensor_msgs
  cv_bridge
  image_transport
  message_generation
  robot_state_publisher
)
find_package( OpenCV 3.4.4 REQUIRED )
# find_package( realsense2 REQUIRED )

# 设定 .cmake 文件存放路径
LIST( APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules )

if( CMAKE_BUILD_TYPE STREQUAL "Debug" )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -Wall -Wno-unused-variable -pthread -fopenmp")
else( CMAKE_BUILD_TYPE STREQUAL "Release" )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -O3 -pthread -Wno-unused-variable")
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -march=native -O3 -pthread -fopenmp -Wno-unused-variable")
endif( CMAKE_BUILD_TYPE STREQUAL "Debug" )
message("Build Type: " ${CMAKE_BUILD_TYPE} ${CMAKE_CXX_FLAGS})

# 在这个地方寻找库文件
LINK_DIRECTORIES($ENV{HALCONROOT}/lib/$ENV{HALCONARCH}
                 ${PROJECT_SOURCE_DIR}/lib)



## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)


## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend tag for "message_generation"
##   * add a build_depend and a exec_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependency has been pulled in
##     but can be declared for certainty nonetheless:
##     * add a exec_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

## Generate services in the 'srv' folder
add_service_files(
   FILES
   AddTwoInts.srv
   PicPose.srv
   ellipse.srv
 )

## Generate actions in the 'action' folder
# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

## Generate added messages and services with any dependencies listed here
generate_messages(
   DEPENDENCIES
   std_msgs
   sensor_msgs
 )

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a exec_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES pic_service
#  CATKIN_DEPENDS roscpp rospy std_msgs
#  DEPENDS system_lib
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations


# 设定库文件编译路径
SET( LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)


# Find Protobuf installation
# Looks for protobuf-config.cmake file installed by Protobuf's cmake installation.
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${protobuf_VERSION}")

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)

# Find gRPC installation
# Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

set(_GRPC_GRPCPP_UNSECURE gRPC::grpc++_unsecure)
set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)

# Proto file
get_filename_component(jk_proto "src/jaka.proto" ABSOLUTE)
get_filename_component(jk_proto_path "${jk_proto}" PATH)

# Generated sources
set(_gRPC_PROTO_GENS_DIR ${CMAKE_CURRENT_BINARY_DIR}/gens)
file(MAKE_DIRECTORY ${_gRPC_PROTO_GENS_DIR})

set(jk_proto_srcs "${_gRPC_PROTO_GENS_DIR}/jaka.pb.cc")
set(jk_proto_hdrs "${_gRPC_PROTO_GENS_DIR}/jaka.pb.h")
set(jk_grpc_srcs "${_gRPC_PROTO_GENS_DIR}/jaka.grpc.pb.cc")
set(jk_grpc_hdrs "${_gRPC_PROTO_GENS_DIR}/jaka.grpc.pb.h")

message(
      OUTPUT "${jk_proto_srcs}" "${jk_proto_hdrs}" "${jk_grpc_srcs}" "${jk_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${_gRPC_PROTO_GENS_DIR}"
        --cpp_out "${_gRPC_PROTO_GENS_DIR}"
        -I "${jk_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${jk_proto}"
      DEPENDS "${jk_proto}")
add_custom_command(
      OUTPUT "${jk_proto_srcs}" "${jk_proto_hdrs}" "${jk_grpc_srcs}" "${jk_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${_gRPC_PROTO_GENS_DIR}"
        --cpp_out "${_gRPC_PROTO_GENS_DIR}"
        -I "${jk_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${jk_proto}"
      DEPENDS "${jk_proto}")

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  include/pic_service
  ../asr_halcon_bridge/include/asr_halcon_bridge
  $ENV{HALCONROOT}/include
  /opt/halcon/include
  /opt/halcon/include/halconcpp
  ${PROJECT_SOURCE_DIR}/include
  ${_gRPC_PROTO_GENS_DIR}
)

message(------------- ${_gRPC_PROTO_GENS_DIR})

set(DEPENDENCIES
    realsense2
    ${OpenCV_LIBS}
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF})

add_library(robot SHARED
    include/robot/robot.h
    src/cubicSpline.cpp
    src/robot.cpp
    src/robot_client.cc
    include/robot/robot_client.h
    ${jk_proto_srcs}
    ${jk_proto_hdrs}
    ${jk_grpc_srcs}
    ${jk_grpc_hdrs}
    )

## Declare a C++ library
#add_library(camera SHARED
   #include/pic_service/realsense.h
 #)

## Add cmake target dependencies of the library
## as an example, code may need to be generated before libraries
## either from message generation or dynamic reconfigure
# add_dependencies(${PROJECT_NAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide

add_executable(robot_operate src/robot_operate.cpp)
target_link_libraries(robot_operate
    robot
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF} 
    ${catkin_LIBRARIES} 
    ${OpenCV_LIBS} 
    ${realsense2_LIBRARY}
    ${DEPENDENCIES})
add_dependencies(robot_operate ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(joint_state_pub src/joint_state_pub.cpp)
target_link_libraries(joint_state_pub
    robot
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF} 
    ${catkin_LIBRARIES} 
    ${OpenCV_LIBS} 
    ${realsense2_LIBRARY}
    ${DEPENDENCIES})
add_dependencies(joint_state_pub ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(js_pub src/js_pub.cpp)
target_link_libraries(js_pub
    robot
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF} 
    ${catkin_LIBRARIES} 
    ${OpenCV_LIBS} 
    ${realsense2_LIBRARY}
    ${DEPENDENCIES})
add_dependencies(js_pub ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(robot_test src/robot_test.cpp)
target_link_libraries(robot_test
    robot
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF} 
    ${catkin_LIBRARIES} 
    ${OpenCV_LIBS} 
    ${realsense2_LIBRARY}
    ${DEPENDENCIES})
add_dependencies(robot_operate ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
## Rename C++ executable without prefix
## The above recommended prefix causes long target names, the following renames the
## target back to the shorter version for ease of user use
## e.g. "rosrun someones_pkg node" instead of "rosrun someones_pkg someones_pkg_node"
# set_target_properties(${PROJECT_NAME}_node PROPERTIES OUTPUT_NAME node PREFIX "")

## Add cmake target dependencies of the executable
## same as for the library above


## Specify libraries to link a library or executable target against


#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_pic_service.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
