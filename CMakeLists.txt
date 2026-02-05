cmake_minimum_required(VERSION 3.2.0)

project(mavlink_sitl_gazebo)

set(CATKIN_DEPENDENCIES
  roscpp
  std_msgs
  gazebo_plugins
  gazebo_ros
  geometry_msgs
  nav_msgs
  sensor_msgs
  )

find_package(catkin REQUIRED COMPONENTS
  ${CATKIN_DEPENDENCIES}
  )

find_package(gazebo REQUIRED)

find_package(OpenCV REQUIRED)

find_package(Boost 1.58 REQUIRED COMPONENTS system thread filesystem)

find_package(mavlink REQUIRED)

find_package(Eigen3 REQUIRED)
set(Eigen_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIRS})
set(Eigen_LIBRARIES ${Eigen_LIBRARIES})

set(PROTOBUF_IMPORT_DIRS "")
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(PROTOBUF_IMPORT_DIRS "${ITR}/gazebo/msgs/proto")
  endif()
endforeach()

set(GAZEBO_MSG_INCLUDE_DIRS)
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(GAZEBO_MSG_INCLUDE_DIRS "${ITR}/gazebo/msgs")
  endif()
endforeach()

# PROTOBUF_IMPORT_DIRS has to be set before
find_package(Protobuf REQUIRED)

set(mav_msgs
  msgs/CommandMotorSpeed.proto
  msgs/MotorSpeed.proto
  )

set(nav_msgs
  msgs/Odometry.proto
  )

set(physics_msgs
    msgs/Force.proto
    msgs/Wind.proto
    )

set(std_msgs msgs/Int32.proto)

set(sensor_msgs
  msgs/Airspeed.proto
  msgs/Imu.proto
  msgs/IRLock.proto
  msgs/Float.proto
  msgs/Groundtruth.proto
  msgs/Range.proto
  msgs/SITLGps.proto
  msgs/MagneticField.proto
  msgs/OpticalFlow.proto
  msgs/Pressure.proto
  )

PROTOBUF_GENERATE_CPP(MAV_PROTO_SRCS MAV_PROTO_HDRS ${mav_msgs})
PROTOBUF_GENERATE_CPP(NAV_PROTO_SRCS NAV_PROTO_HDRS ${nav_msgs})
PROTOBUF_GENERATE_CPP(PHY_PROTO_SRCS PHY_PROTO_HDRS ${physics_msgs})
PROTOBUF_GENERATE_CPP(STD_PROTO_SRCS STD_PROTO_HDRS ${std_msgs})
PROTOBUF_GENERATE_CPP(SEN_PROTO_SRCS SEN_PROTO_HDRS ${sensor_msgs})

set(CMAKE_C_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(LIBRARIES
  # used by MRS
  gazebo_gps_plugin
  gazebo_motor_model
  gazebo_multirotor_base_plugin
  gazebo_mavlink_interface
  gazebo_magnetometer_plugin
  gazebo_groundtruth_plugin
  gazebo_barometer_plugin
  gazebo_imu_plugin
  gazebo_wind_plugin
  gazebo_lidar_plugin

  # are needed as dependencies
  gazebo_opticalflow_mockup_plugin
  gazebo_opticalflow_plugin

  # MRS does not use
  # gazebo_airspeed_plugin
  # gazebo_irlock_plugin
  # gazebo_sonar_plugin
  # gazebo_uuv_plugin
  # gazebo_vision_plugin
  # gazebo_controller_interface
  # gazebo_drop_plugin
  )

add_subdirectory(external/OpticalFlow OpticalFlow)
set(OpticalFlow_LIBS "OpticalFlow")

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS ${CATKIN_DEPENDENCIES}
  DEPENDS GAZEBO Boost Eigen OpenCV
  LIBRARIES ${LIBRARIES}
  )

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  include
  ${Boost_INCLUDE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${Eigen_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
  ${GAZEBO_MSG_INCLUDE_DIRS}
  ${mavlink_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${OpticalFlow_INCLUDE_DIRS}
  )

link_directories(${GAZEBO_LIBRARY_DIRS})

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

link_libraries(
  ${Boost_SYSTEM_LIBRARY_RELEASE}
  ${Boost_THREAD_LIBRARY_RELEASE}
  ${Boost_TIMER_LIBRARY_RELEASE}
  ${GAZEBO_LIBRARIES}
  ${OpenCV_LIBRARIES}
  )

link_directories(
  ${GAZEBO_LIBRARY_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
  )

add_library(mav_msgs SHARED ${MAV_PROTO_SRCS})
add_library(nav_msgs SHARED ${NAV_PROTO_SRCS})
add_library(physics_msgs SHARED ${PHY_PROTO_SRCS})
add_library(std_msgs SHARED ${STD_PROTO_SRCS})
add_library(sensor_msgs SHARED ${SEN_PROTO_SRCS})

link_libraries(mav_msgs nav_msgs std_msgs sensor_msgs)
link_libraries(physics_msgs)

# user by MRS
add_library(gazebo_gps_plugin SHARED src/gazebo_gps_plugin.cpp)
add_library(gazebo_groundtruth_plugin SHARED src/gazebo_groundtruth_plugin.cpp)
add_library(gazebo_lidar_plugin SHARED src/gazebo_lidar_plugin.cpp)
add_library(gazebo_imu_plugin SHARED src/gazebo_imu_plugin.cpp)
add_library(gazebo_mavlink_interface SHARED src/gazebo_mavlink_interface.cpp src/mavlink_interface.cpp)
add_library(gazebo_motor_model SHARED src/gazebo_motor_model.cpp)
add_library(gazebo_multirotor_base_plugin SHARED src/gazebo_multirotor_base_plugin.cpp)
add_library(gazebo_wind_plugin SHARED src/gazebo_wind_plugin.cpp)
add_library(gazebo_magnetometer_plugin SHARED src/gazebo_magnetometer_plugin.cpp src/geo_mag_declination.cpp)
add_library(gazebo_barometer_plugin SHARED src/gazebo_barometer_plugin.cpp)

# not used but are dependencies
add_library(gazebo_opticalflow_mockup_plugin SHARED src/gazebo_opticalflow_mockup_plugin.cpp)
add_library(gazebo_opticalflow_plugin SHARED src/gazebo_opticalflow_plugin.cpp)

# not used by MRS
# add_library(gazebo_drop_plugin SHARED src/gazebo_drop_plugin.cpp)
# add_library(gazebo_controller_interface SHARED src/gazebo_controller_interface.cpp)
# add_library(gazebo_sonar_plugin SHARED src/gazebo_sonar_plugin.cpp)
# add_library(gazebo_irlock_plugin SHARED src/gazebo_irlock_plugin.cpp)
# add_library(gazebo_airspeed_plugin SHARED src/gazebo_airspeed_plugin.cpp)
# add_library(gazebo_vision_plugin SHARED src/gazebo_vision_plugin.cpp)
# add_library(gazebo_uuv_plugin SHARED src/gazebo_uuv_plugin.cpp)

foreach(target ${LIBRARIES})
  target_link_libraries(${target} ${Boost_LIBRARIES} ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES})
endforeach()

target_link_libraries(gazebo_opticalflow_plugin ${OpticalFlow_LIBS})

foreach(target ${LIBRARIES})
  add_dependencies(${target} mav_msgs nav_msgs std_msgs sensor_msgs)
  add_dependencies(${target} physics_msgs)
endforeach()

## --------------------------------------------------------------
## |                           install                          |
## --------------------------------------------------------------

install(TARGETS ${LIBRARIES} physics_msgs mav_msgs nav_msgs std_msgs sensor_msgs
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
  )
