cmake_minimum_required(VERSION 3.5)
project(orbslam3)

# You should set the PYTHONPATH to your own python site-packages path
set(ENV{PYTHONPATH} "/opt/ros/humble/lib/python3.10/site-packages/")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)
set(CMAKE_PREFIX_PATH "/home/oficina-robotica/ORB_SLAM3_2/Thirdparty/libtorch")

# Check C++17 support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
if(COMPILER_SUPPORTS_CXX17)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
   add_definitions(-DCOMPILEDWITHC17)
   message(STATUS "Using flag -std=c++17.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(message_filters REQUIRED)
find_package(Sophus REQUIRED)
find_package(Pangolin REQUIRED)
find_package(ORB_SLAM3 REQUIRED)
find_package(GLUT REQUIRED)
find_package(OpenGL REQUIRED)
find_package(Torch REQUIRED)
#import cuda
find_package(CUDA REQUIRED)
set(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda-11.8)
set(Torch_DIR "/home/oficina-robotica/ORB_SLAM3_2/Thirdparty/libtorch/share/cmake/Torch")

include_directories(
  include
  ${ORB_SLAM3_ROOT_DIR}/include
  ${ORB_SLAM3_ROOT_DIR}/include/CameraModels
  ${GLUT_INCLUDE_DIRS}
  ${TORCH_INCLUDE_DIRS}
  ${CUDA_INCLUDE_DIRS}
)

link_directories(
  include
)


add_executable(mono
  src/monocular/mono.cpp
  src/monocular/monocular-slam-node.cpp
)
ament_target_dependencies(mono rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin OpenCV)
target_link_libraries(mono ${GLUT_LIBRARIES} ${OpenGL_LIBRARIES} ${TORCH_LIBRARIES})

add_executable(rgbd
  src/rgbd/rgbd.cpp
  src/rgbd/rgbd-slam-node.cpp
)
ament_target_dependencies(rgbd rclcpp sensor_msgs cv_bridge message_filters ORB_SLAM3 Pangolin OpenCV)
target_link_libraries(rgbd ${GLUT_LIBRARIES} ${OpenGL_LIBRARIES} ${TORCH_LIBRARIES})

add_executable(stereo
  src/stereo/stereo.cpp
  src/stereo/stereo-slam-node.cpp
)
ament_target_dependencies(stereo rclcpp sensor_msgs cv_bridge message_filters ORB_SLAM3 Pangolin OpenCV )
target_link_libraries(stereo ${GLUT_LIBRARIES} ${OpenGL_LIBRARIES} ${TORCH_LIBRARIES})


add_executable(stereo-inertial
  src/stereo-inertial/stereo-inertial.cpp
  src/stereo-inertial/stereo-inertial-node.cpp
)
ament_target_dependencies(stereo-inertial rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin OpenCV)
target_link_libraries(stereo-inertial ${GLUT_LIBRARIES} ${OpenGL_LIBRARIES} ${TORCH_LIBRARIES})

add_executable(stereo-gazebo
  src/stereo-gazebo/stereo-gazebo.cpp
  src/stereo-gazebo/stereo-gazebo-node.cpp
)
ament_target_dependencies(stereo-gazebo rclcpp sensor_msgs cv_bridge message_filters ORB_SLAM3 Pangolin OpenCV)
target_link_libraries(stereo-gazebo ${GLUT_LIBRARIES} ${OpenGL_LIBRARIES} ${TORCH_LIBRARIES})


install(TARGETS mono rgbd stereo stereo-inertial stereo-gazebo
  DESTINATION lib/${PROJECT_NAME})

# Install launch files.
#install(DIRECTORY launch config vocabulary
#  DESTINATION share/${PROJECT_NAME}/)

ament_package()

